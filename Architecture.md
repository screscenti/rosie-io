# Rosie System Architecture

**Version:** 0.1.0  
**Last Updated:** January 2026

---

## 1. Overview

Rosie is a **network-native voice assistant** with a central server coordinating multiple distributed endpoints. Unlike monolithic systems, Rosie separates compute (server) from interaction (clients), enabling flexible deployment and efficient resource usage.

**Key Principles:**
- **Server/Client Separation:** Heavy processing on server, lightweight clients
- **Tiered AI Architecture:** Local → Network → Cloud routing based on query complexity
- **Modular Design:** Swap components without rewriting core
- **Real-time Communication:** WebSocket + WebRTC for low-latency audio streaming

---

## 2. High-Level Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                      USER'S NETWORK                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              ROSIE SERVER (Central Brain)                  │ │
│  ├────────────────────────────────────────────────────────────┤ │
│  │                                                            │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐  │ │
│  │  │   FastAPI    │  │   Ollama     │  │ Faster-Whisper │  │ │
│  │  │   Backend    │  │  (3-4B LLM)  │  │   (STT)        │  │ │
│  │  └──────────────┘  └──────────────┘  └────────────────┘  │ │
│  │                                                            │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐  │ │
│  │  │  Piper TTS   │  │  ChromaDB    │  │  Integration   │  │ │
│  │  │              │  │  (Memory)    │  │      Hub       │  │ │
│  │  └──────────────┘  └──────────────┘  └────────────────┘  │ │
│  │                                                            │ │
│  │  ┌────────────────────────────────────────────────────┐  │ │
│  │  │             WebSocket + WebRTC Server              │  │ │
│  │  └────────────────────────────────────────────────────┘  │ │
│  │                          ▲                                │ │
│  └──────────────────────────┼────────────────────────────────┘ │
│                             │                                   │
│         ┌───────────────────┼───────────────────┐              │
│         │                   │                   │              │
│    ┌────▼─────┐       ┌────▼─────┐       ┌────▼─────┐        │
│    │  Echo    │       │ Raspberry│       │   Web    │        │
│    │  Show    │       │    Pi    │       │ Browser  │        │
│    │(Endpoint)│       │(Endpoint)│       │(Endpoint)│        │
│    └──────────┘       └──────────┘       └──────────┘        │
│                                                                 │
│  Optional: Network AI Server                                   │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  GPU Server (e.g., "Sofia")                            │   │
│  │  • Ollama (70B+ models)                                │   │
│  │  • NVIDIA RTX 3060/3090                                │   │
│  └────────────────────────────────────────────────────────┘   │
│                          ▲                                      │
└──────────────────────────┼──────────────────────────────────────┘
                           │
                    Optional Cloud
                  (Gemini, OpenAI, Grok)
```

---

## 3. Component Architecture

### 3.1 Rosie Server Components

#### 3.1.1 API Layer (FastAPI)
**Responsibilities:**
- Expose REST API for configuration and control
- Handle WebSocket connections for real-time client communication
- Coordinate WebRTC streams for audio
- Authentication and authorization (future)
- Rate limiting and request validation

**Key Endpoints:**
- `GET /api/v1/health` - Health check
- `GET /api/v1/status` - System status (connected clients, resource usage)
- `POST /api/v1/config` - Update configuration
- `WS /api/v1/stream` - WebSocket for client connection
- `POST /api/v1/memory/query` - Query conversation history

#### 3.1.2 STT Service (Faster-Whisper)
**Responsibilities:**
- Receive audio stream from clients via WebRTC
- Transcribe speech to text in real-time
- Support streaming transcription (partial results)
- Handle multiple concurrent streams

**Implementation:**
- Faster-Whisper library (CTranslate2 optimized)
- Model: `base.en` (default) or `small.en` for better accuracy
- GPU acceleration if available (CUDA or ROCm)
- VAD (Voice Activity Detection) to reduce processing

**Performance Targets:**
- Latency: <500ms from speech end to transcription
- Accuracy: >98% WER on clean speech
- Throughput: 5+ concurrent streams on 6-core CPU

#### 3.1.3 LLM Service (Ollama Client)
**Responsibilities:**
- Route queries to appropriate LLM tier (local/network/cloud)
- Stream responses word-by-word to clients
- Manage conversation context
- Handle function calling / tool use

**Tiered Routing Logic:**
```python
def route_query(query: str, context: dict) -> LLMTier:
    # Check user preference first
    if user_preference == "always_local":
        return LLMTier.LOCAL
    
    # Classify query complexity
    complexity = classify_complexity(query)
    
    if complexity == "simple":  # Facts, time, weather
        return LLMTier.LOCAL
    elif complexity == "medium" and network_gpu_available():
        return LLMTier.NETWORK
    elif complexity == "complex":
        return LLMTier.CLOUD if cloud_enabled() else LLMTier.NETWORK
    
    # Fallback
    return LLMTier.LOCAL
```

**Local Tier:**
- Model: Llama 3.2 3B or Phi-3 Mini (3.8B)
- Use cases: Quick facts, simple commands, small talk
- Target: <1s response time

**Network Tier (Optional):**
- Model: Llama 3.3 70B (on GPU server like Sofia)
- Use cases: Complex reasoning, long-form content, analysis
- Target: <3s response time

**Cloud Tier (Optional):**
- APIs: Gemini 2.0, GPT-4o, Grok-2
- Use cases: Fallback, specialized tasks, latest models
- Target: <5s response time

#### 3.1.4 TTS Service (Piper)
**Responsibilities:**
- Convert text responses to speech audio
- Stream audio back to client
- Support multiple voices/languages (future)
- Handle SSML for pronunciation control

**Implementation:**
- Piper TTS (fast, high-quality)
- Model: `en_US-lessac-medium` (default)
- Streaming output (start playback before full text processed)
- Cache common phrases for instant playback

**Performance Targets:**
- Latency: <300ms from text to audio start
- Quality: Natural-sounding, minimal artifacts
- Throughput: 5+ concurrent synthesis requests

#### 3.1.5 Memory Service (ChromaDB + SQLite)
**Responsibilities:**
- Store conversation history
- Embed messages for semantic search
- Retrieve relevant context for LLM queries
- Support memory management (clear, export, tag)

**Data Model:**
```
Conversations (SQLite)
├── id (UUID)
├── user_id (optional, future)
├── room_id (identifies which client/room)
├── timestamp
├── speaker ("user" or "rosie")
├── text (original message)
├── embedding_id (ref to ChromaDB)
└── metadata (JSON: intent, tools_used, etc.)

ChromaDB Collections
├── conversations (embeddings of messages)
├── documents (ingested PDFs, notes, etc.)
└── facts (extracted entities, preferences)
```

**RAG Flow:**
```python
async def generate_response(query: str, context: dict):
    # 1. Retrieve relevant past conversations
    relevant_memories = await memory.search(
        query=query,
        k=5,  # Top 5 most relevant
        filters={"room_id": context["room_id"]}
    )
    
    # 2. Build prompt with context
    prompt = build_prompt(
        query=query,
        conversation_history=context["recent"],  # Last 10 messages
        retrieved_memories=relevant_memories,
        system_prompt=config.system_prompt
    )
    
    # 3. Generate response
    response = await llm.generate(prompt)
    
    # 4. Store interaction
    await memory.store(query, response, context)
    
    return response
```

#### 3.1.6 Integration Hub
**Responsibilities:**
- Provide unified interface for external integrations
- Handle authentication for third-party APIs
- Map voice commands to integration actions
- Plugin architecture for community extensions

**Core Integrations (Phase 3):**
- **Home Assistant:** REST API + WebSocket for real-time state
- **Google Calendar:** OAuth2 + Calendar API v3
- **Monica CRM:** REST API (self-hosted)
- **Mattermost:** Webhooks + Bot API

**Plugin Interface:**
```python
class IntegrationPlugin(ABC):
    @abstractmethod
    async def initialize(self, config: dict):
        """Set up integration with API keys, etc."""
        pass
    
    @abstractmethod
    async def handle_command(self, intent: str, entities: dict) -> dict:
        """Execute command and return result"""
        pass
    
    @abstractmethod
    async def get_status(self) -> dict:
        """Return integration health/status"""
        pass
```

**Example Flow (Home Assistant):**
```
User: "Turn on the office lights"
  ↓
STT → "Turn on the office lights"
  ↓
LLM (with function calling):
  Function: home_assistant.turn_on_light
  Arguments: {"room": "office", "entity": "light.office_ceiling"}
  ↓
Integration Hub → Home Assistant API
  ↓
Response: {"success": true, "state": "on"}
  ↓
TTS: "Office lights are on"
```

---

### 3.2 Client Endpoint Architecture

#### 3.2.1 Generic Web Client
**Technology:** HTML/CSS/JS (vanilla or lightweight framework)

**Components:**
- **WebRTC Manager:** Capture mic, stream audio to server
- **WebSocket Manager:** Send/receive control messages
- **UI Renderer:** Display transcription, responses, status
- **Audio Player:** Play TTS audio from server

**Responsibilities:**
- Handle audio I/O (getUserMedia API)
- Maintain WebSocket connection
- Display visual feedback (listening, thinking, speaking)
- Support touch/click interaction (manual input, mute, etc.)

**Deployment:**
- Served by Rosie server (embedded in FastAPI)
- Accessible at `http://rosie.local:8000`
- Responsive design (works on tablets, desktop)

#### 3.2.2 Echo Show Client (LineageOS)
**Technology:** Android WebView (Chromium) in kiosk mode

**Setup:**
1. Flash LineageOS 14.1 on Echo Show Gen 1
2. Install F-Droid + Chromium or Fully Kiosk Browser
3. Configure auto-start to `http://rosie.local:8000`
4. Disable sleep, enable always-on display

**Optimizations:**
- Use Echo Show's 7-mic array for far-field capture
- Leverage hardware audio processing
- Display ambient animations when idle
- Support tap-to-wake (optional if wake word disabled)

#### 3.2.3 Raspberry Pi Client
**Technology:** Minimal Linux + Chromium kiosk

**Setup:**
1. Install Raspberry Pi OS Lite
2. Install X11 + Openbox + Chromium
3. Auto-start Chromium in kiosk mode
4. Attach USB mic + speaker or HAT

**Hardware Recommendations:**
- Raspberry Pi 4 (4GB) or Pi 5
- ReSpeaker 2-Mics HAT or USB mic
- Mini HDMI display (7" touchscreen)

---

## 4. Data Flow Diagrams

### 4.1 Voice Interaction Flow
```
┌──────────┐
│  User    │ Speaks: "What's the weather?"
└─────┬────┘
      │ Audio Stream (WebRTC)
      ▼
┌──────────────┐
│  Endpoint    │ (Echo Show, Pi, Browser)
│  (Client)    │
└──────┬───────┘
       │ Raw Audio (Opus/PCM)
       ▼
┌────────────────────┐
│  Rosie Server      │
│  STT Service       │ ← Faster-Whisper
└─────┬──────────────┘
      │ Text: "What's the weather?"
      ▼
┌────────────────────┐
│  LLM Routing       │ → Classify: Simple query
└─────┬──────────────┘
      │ Route to: Local LLM
      ▼
┌────────────────────┐
│  Memory Service    │ → Retrieve: User location, preferences
└─────┬──────────────┘
      │ Context: {location: "Ellenton, FL"}
      ▼
┌────────────────────┐
│  Ollama (Local)    │ ← Llama 3.2 3B
│  + Function Call   │ → fetch_weather(location="Ellenton, FL")
└─────┬──────────────┘
      │ Tool Result: {temp: 72, condition: "Sunny"}
      ▼
┌────────────────────┐
│  LLM Response      │ → "It's 72°F and sunny in Ellenton"
└─────┬──────────────┘
      │ Text response
      ▼
┌────────────────────┐
│  TTS Service       │ ← Piper TTS
└─────┬──────────────┘
      │ Audio Stream (WAV/Opus)
      ▼
┌──────────────┐
│  Endpoint    │ Plays audio via speaker
└──────────────┘
```

**Latency Breakdown (Target):**
- Audio transmission: <50ms
- STT processing: <500ms
- LLM inference: <1000ms
- TTS synthesis: <300ms
- Audio playback: <50ms
- **Total: ~1.9s end-to-end**

### 4.2 Integration Command Flow
```
User: "Turn on the living room lights"
  ↓
STT → Text
  ↓
LLM (with function calling enabled)
  ↓
Function: {
  "name": "home_assistant.turn_on",
  "arguments": {
    "entity_id": "light.living_room",
    "brightness": 100
  }
}
  ↓
Integration Hub
  ↓
Home Assistant API
  POST /api/services/light/turn_on
  Body: {"entity_id": "light.living_room", "brightness": 100}
  ↓
Home Assistant executes command
  ↓
Response: {"success": true}
  ↓
LLM generates natural response
  ↓
TTS: "Living room lights are on"
```

---

## 5. Deployment Architecture

### 5.1 Docker Compose Stack
```yaml
version: '3.8'

services:
  rosie-server:
    build: ./server
    ports:
      - "8000:8000"  # HTTP/WebSocket
    volumes:
      - ./data/conversations:/app/data/conversations
      - ./data/chromadb:/app/data/chromadb
      - ./config:/app/config
    environment:
      - OLLAMA_HOST=http://ollama:11434
      - LOG_LEVEL=INFO
    depends_on:
      - ollama
      - chromadb

  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ./data/ollama:/root/.ollama
    # GPU support (NVIDIA)
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - ./data/chromadb:/chroma/chroma

  web-ui:
    build: ./web-ui
    ports:
      - "3000:3000"
    environment:
      - API_URL=http://rosie-server:8000
```

### 5.2 Network Topology
```
┌─────────────────────────────────────────────────┐
│             User's Home Network                  │
│  (192.168.1.0/24)                                │
├─────────────────────────────────────────────────┤
│                                                   │
│  Rosie Server                                     │
│  └─ 192.168.1.100:8000                           │
│     ├─ HTTP/WebSocket (clients)                  │
│     └─ Admin UI (port 3000)                      │
│                                                   │
│  Optional: GPU Server (Sofia)                    │
│  └─ 192.168.1.101:11434                          │
│     └─ Ollama API (large models)                 │
│                                                   │
│  Clients:                                         │
│  ├─ Echo Show (Office) - 192.168.1.110          │
│  ├─ Echo Show (Kitchen) - 192.168.1.111         │
│  └─ Raspberry Pi (Bedroom) - 192.168.1.112      │
│                                                   │
│  External (Optional):                             │
│  └─ Internet → Cloud APIs (Gemini, etc.)        │
└───────────────────────────────────────────────────┘
```

**Reverse Proxy (Optional - Caddy):**
```
rosie.local {
  reverse_proxy localhost:8000
}

admin.rosie.local {
  reverse_proxy localhost:3000
}
```

---

## 6. Security Architecture

### 6.1 Threat Model

**Assets:**
- Conversation history (private data)
- User preferences and memories
- Integration credentials (API keys)
- System configuration

**Threats:**
- Unauthorized access to admin UI
- Man-in-the-middle attacks on client/server communication
- Credential theft from config files
- DoS attacks on voice endpoints

**Mitigations:**
- HTTPS/WSS for all communication (self-signed cert acceptable for local)
- Environment variables for secrets (not committed to git)
- Optional HTTP Basic Auth for admin UI
- Rate limiting on API endpoints
- Firewall rules (expose only necessary ports)

### 6.2 Authentication & Authorization (Future)

**Phase 1 (v0.1):** Single-user, no authentication
- Assumes trusted network
- Admin UI accessible to anyone on LAN

**Phase 2 (v0.5):** Basic authentication
- HTTP Basic Auth for admin UI
- API key for client registration
- Room-based permissions (office client can't control bedroom lights)

**Phase 3 (v1.0):** Multi-user support
- User accounts with separate conversation histories
- Voice biometrics for speaker identification
- Per-user integration credentials

---

## 7. Scalability Considerations

### 7.1 Vertical Scaling (Single Server)
**Bottlenecks:**
- STT: CPU-bound (can use GPU)
- LLM: Memory-bound for large models
- TTS: CPU-bound
- Memory: Disk I/O for ChromaDB

**Optimizations:**
- Queue concurrent requests (Celery/asyncio)
- GPU acceleration where available
- Model quantization (4-bit for 70B models)
- Caching (TTS phrases, LLM responses for common queries)

### 7.2 Horizontal Scaling (Future)
**Architecture:**
```
       Load Balancer
            ↓
    ┌───────┼───────┐
    ▼       ▼       ▼
  Server1 Server2 Server3
    ↓       ↓       ↓
      Shared ChromaDB
      Shared Redis (session state)
```

**Challenges:**
- Session affinity (WebSocket connections)
- Shared memory/state
- Distributed rate limiting

**Not a priority for v1.0** (single-server handles 10-20 concurrent users easily)

---

## 8. Monitoring & Observability

### 8.1 Metrics to Track
- **Performance:**
  - Response latency (STT, LLM, TTS)
  - Queue depth (pending requests)
  - CPU/RAM/GPU usage
- **Reliability:**
  - Uptime
  - Error rates (by component)
  - Client disconnections
- **Usage:**
  - Requests per minute
  - Active clients
  - Most common intents/commands

### 8.2 Logging Strategy
**Structured JSON logs:**
```json
{
  "timestamp": "2026-01-11T12:34:56Z",
  "level": "INFO",
  "component": "stt",
  "room_id": "office",
  "message": "Transcription complete",
  "duration_ms": 487,
  "text": "What's the weather?"
}
```

**Log Levels:**
- DEBUG: Detailed traces (disable in production)
- INFO: Normal operations
- WARN: Recoverable errors
- ERROR: Critical failures

### 8.3 Health Checks
**Endpoints:**
- `GET /health` → 200 OK (always responds if server alive)
- `GET /health/stt` → Check STT service loaded
- `GET /health/llm` → Check Ollama reachable
- `GET /health/memory` → Check ChromaDB writable

**Docker Healthcheck:**
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
```

---

## 9. Failure Modes & Recovery

### 9.1 Component Failures

**STT Service Down:**
- Impact: Cannot transcribe new audio
- Detection: Health check fails
- Recovery: Restart container, use backup STT (cloud API)
- User Experience: "Sorry, I'm having trouble hearing right now"

**LLM Local Down:**
- Impact: Cannot process queries locally
- Detection: Ollama unreachable
- Recovery: Fall back to network/cloud tier
- User Experience: Slight latency increase

**Memory Service Down:**
- Impact: No conversation history, no RAG
- Detection: ChromaDB connection fails
- Recovery: Restart ChromaDB, use temp in-memory store
- User Experience: "I'm having trouble remembering past conversations"

**Network Partition:**
- Impact: Clients disconnected from server
- Detection: WebSocket disconnect
- Recovery: Clients auto-reconnect with exponential backoff
- User Experience: "Reconnecting..." message on client

### 9.2 Data Loss Prevention

**Critical Data:**
- Conversation history (SQLite + ChromaDB)
- Configuration files
- Integration credentials

**Backup Strategy:**
- Daily backup of `/data` volume
- Export conversations to JSON (manual trigger)
- Config files in git (excluding secrets)

---

## 10. Evolution & Extensibility

### 10.1 Plugin Architecture (Future)
**Goal:** Allow community to add integrations without modifying core

**Design:**
```python
# plugins/my_integration/plugin.py
from rosie.plugins import IntegrationPlugin

class MyIntegration(IntegrationPlugin):
    name = "my_integration"
    version = "1.0.0"
    
    async def initialize(self, config):
        self.api_key = config["api_key"]
    
    async def handle_command(self, intent, entities):
        if intent == "do_something":
            result = await self.api_call(entities)
            return {"success": True, "message": result}
    
    def get_intents(self):
        return ["do_something", "do_another_thing"]
```

**Loading:**
```python
# Load plugins from /app/plugins directory
for plugin_dir in Path("/app/plugins").iterdir():
    plugin = load_plugin(plugin_dir / "plugin.py")
    register_plugin(plugin)
```

### 10.2 Model Swapping
**Goal:** Easily swap STT/TTS/LLM models

**Configuration:**
```yaml
# config.yaml
stt:
  engine: faster-whisper  # or: whisper-api, vosk
  model: base.en
  device: cuda  # or: cpu

tts:
  engine: piper  # or: espeak, coqui-tts
  voice: en_US-lessac-medium

llm:
  local:
    provider: ollama  # or: llama.cpp, transformers
    model: llama3.2:3b
  network:
    provider: ollama
    host: http://sofia.local:11434
    model: llama3.3:70b
  cloud:
    provider: gemini  # or: openai, anthropic
    api_key: ${GEMINI_API_KEY}
    model: gemini-2.0-flash
```

---

## 11. Technology Decisions Rationale

### 11.1 Why FastAPI?
- **Async-first:** Perfect for WebSocket/WebRTC
- **Type hints:** Better code quality, IDE support
- **Auto docs:** OpenAPI/Swagger out of the box
- **Fast:** Comparable to Node.js/Go performance
- **Ecosystem:** Great Python AI/ML library support

### 11.2 Why Ollama for LLM?
- **Easy deployment:** Single binary, Docker image
- **Model management:** Pull models like Docker images
- **GPU support:** CUDA and ROCm out of box
- **API compatibility:** OpenAI-compatible API
- **Community:** Active development, good docs

### 11.3 Why Faster-Whisper for STT?
- **Performance:** 4x faster than Whisper (CTranslate2)
- **Accuracy:** Same as OpenAI Whisper
- **Local:** No cloud dependency
- **Streaming:** Partial results possible
- **GPU:** CUDA/ROCm support

### 11.4 Why Piper for TTS?
- **Speed:** Real-time on CPU
- **Quality:** Natural-sounding (better than eSpeak)
- **Lightweight:** Small models (<100MB)
- **Offline:** No cloud dependency
- **SSML:** Pronunciation control support

### 11.5 Why ChromaDB for Memory?
- **Simple:** SQLite-based, no separate server (optional)
- **Embeddings:** Built-in vector search
- **Python-native:** Easy integration
- **Lightweight:** Low resource usage
- **Filters:** Metadata filtering for room-specific queries

---

## 12. Comparison to Alternatives

| Feature | Rosie | Mycroft | Rhasspy | Home Assistant Voice |
|---------|-------|---------|---------|---------------------|
| **Architecture** | Network-first | Monolithic | Monolithic | HA-integrated |
| **Modern Stack** | ✅ Python 3.11+ | ❌ Python 2/3 mix | ❌ Python 3.7 | ✅ Python 3.11+ |
| **Local LLM** | ✅ Ollama | ❌ Intent-only | ❌ Intent-only | ✅ (via add-on) |
| **Admin UI** | ✅ Web | ❌ CLI/YAML | ❌ Web (basic) | ✅ HA UI |
| **Docker-native** | ✅ | ⚠️ Partial | ⚠️ Partial | ✅ |
| **Multi-endpoint** | ✅ Designed for | ❌ Single device | ❌ Single device | ⚠️ Requires ESPHome |
| **Active Development** | ✅ | ⚠️ Slow | ⚠️ Slow | ✅ |

**Rosie's Unique Value:**
1. **Network-first** (not an afterthought)
2. **Modern Python stack** (not legacy)
3. **Tiered AI** (local/network/cloud routing)
4. **Admin UI** (not YAML configuration)
5. **Extensible** (plugin architecture planned)

---

## End of Architecture Document

**Next Steps:**
1. Review and approve architecture
2. Create detailed API specification (API.md)
3. Begin Phase 1 implementation (core voice loop)
