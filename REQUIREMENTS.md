# Rosie Requirements Specification

**Version:** 0.1.0  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. Project Goals

### 1.1 Core Philosophy
- **Privacy-First:** All default processing happens locally with no cloud dependencies
- **Modular Architecture:** Swap components (LLM, STT, TTS, integrations) without system redesign
- **Network-Native:** Central server with distributed endpoints (not monolithic)
- **Resource-Efficient:** Run on consumer hardware (no datacenter required)
- **Open Ecosystem:** Support multiple endpoint types and integration patterns
- **Self-Hosted:** Users maintain full control over their data and infrastructure

### 1.2 Target Audience
- Homelab enthusiasts running self-hosted services
- Privacy-conscious users avoiding cloud assistants
- Smart home tinkerers with existing automation infrastructure
- Self-hosting community members (Docker, Linux-savvy)
- Developers wanting extensible voice assistant platform

### 1.3 Success Criteria
- **Performance:** <200ms response time for local queries
- **Reliability:** 99.9% uptime for core voice loop
- **Resource Usage:** <2GB RAM idle, <10% CPU on 6-core system
- **Ease of Deploy:** Working system in <30 minutes with Docker Compose
- **Extensibility:** Add new integration without modifying core code

---

## 2. System Architecture Requirements

### 2.1 High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                      User's Network                          │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────┐           │
│  │         ROSIE SERVER (Central Brain)         │           │
│  ├──────────────────────────────────────────────┤           │
│  │ • FastAPI Backend (WebSocket + WebRTC)       │           │
│  │ • Local LLM (Ollama - 3-4B models)          │           │
│  │ • STT Engine (Faster-Whisper)                │           │
│  │ • TTS Engine (Piper)                         │           │
│  │ • Memory/RAG (ChromaDB)                      │           │
│  │ • Integration Hub (Home Assistant, etc.)     │           │
│  │ • Admin Web UI                               │           │
│  └──────────────────────────────────────────────┘           │
│           │                │                 │               │
│           │                │                 │               │
│  ┌────────▼───┐   ┌───────▼────┐   ┌───────▼────┐         │
│  │ Echo Show  │   │ Raspberry  │   │   Web      │         │
│  │ (LineageOS)│   │ Pi Display │   │  Browser   │         │
│  └────────────┘   └────────────┘   └────────────┘         │
│                                                               │
│  Optional Network AI Server:                                 │
│  ┌──────────────────────────────────────────────┐           │
│  │  GPU Server (e.g., "Sofia")                  │           │
│  │  • Ollama (70B+ models)                      │           │
│  │  • High-performance inference                │           │
│  └──────────────────────────────────────────────┘           │
│                          │                                    │
└──────────────────────────┼────────────────────────────────────┘
                           │
                  Optional Cloud APIs
                  (Gemini, OpenAI, Grok)
```

### 2.2 Component Requirements

#### 2.2.1 Rosie Server (Core)
**MUST:**
- Run in Docker container(s) for portability
- Support headless operation (no GUI required on server)
- Expose WebSocket API for real-time client communication
- Expose REST API for configuration and control
- Support multiple simultaneous client connections
- Maintain conversation context across sessions
- Log all interactions for debugging
- Support configuration via environment variables and config files

**SHOULD:**
- Auto-restart on failure
- Support hot-reload for development
- Provide health check endpoints
- Support graceful shutdown

#### 2.2.2 Client Endpoints
**MUST:**
- Stream audio to server via WebRTC or WebSocket
- Display transcription in real-time
- Play TTS audio from server
- Support touch/click interaction
- Auto-reconnect on network disruption
- Identify themselves to server (room name, capabilities)

**SHOULD:**
- Support offline mode (basic functionality without server)
- Display visual feedback (listening, processing, speaking states)
- Support wake word detection locally (optional)

---

## 3. Functional Requirements

### 3.1 Voice Interaction

#### 3.1.1 Speech-to-Text (STT)
**MUST:**
- Support real-time streaming transcription
- Handle multiple languages (starting with English)
- Work with microphone array or single mic
- Support punctuation and capitalization
- Achieve <2% word error rate on clear speech

**SHOULD:**
- Support wake word detection ("Hey Rosie")
- Handle background noise gracefully
- Support voice activity detection (VAD)
- Cache models for offline operation

**Implementation:** Faster-Whisper (local inference)

#### 3.1.2 Text-to-Speech (TTS)
**MUST:**
- Generate natural-sounding speech
- Support adjustable speed and pitch
- Latency <500ms from text to audio start
- Support SSML for pronunciation control

**SHOULD:**
- Support multiple voices/personas
- Support emotional tone modulation
- Cache common phrases

**Implementation:** Piper TTS (local, fast)

#### 3.1.3 Large Language Model (LLM)
**MUST:**
- Support tiered inference architecture:
  1. **Local (Default):** 3-4B parameter model on Rosie server
  2. **Network (Optional):** 70B+ model on GPU server
  3. **Cloud (Optional):** Gemini, OpenAI, Grok APIs

**Routing Logic:**
- Simple queries (time, weather, facts) → Local
- Complex reasoning (analysis, long-form) → Network or Cloud
- User configurable priority and fallback

**MUST Support:**
- Streaming responses (word-by-word output)
- Conversation context (multi-turn)
- Function calling / tool use
- Configurable system prompts

**Implementation:** Ollama client with routing logic

### 3.2 Memory & Context

#### 3.2.1 Conversation Memory
**MUST:**
- Store conversation history per user/session
- Support semantic search across past conversations
- Retain context within session
- Support manual memory management (clear, export)

**SHOULD:**
- Summarize old conversations automatically
- Support memory import/export (JSON format)
- Tag conversations by topic/room
- Support shared vs private memory contexts

**Implementation:** ChromaDB for vector embeddings + SQLite for metadata

#### 3.2.2 RAG (Retrieval Augmented Generation)
**MUST:**
- Index documents for semantic search
- Retrieve relevant context for LLM queries
- Support PDF, TXT, Markdown ingestion

**SHOULD:**
- Support Google Drive integration
- Support live web search
- Auto-update indices on document changes

### 3.3 Integrations

#### 3.3.1 Home Automation
**MUST (Priority 1):**
- Home Assistant integration (REST API + WebSocket)
  - Control lights, switches, scenes
  - Query sensor states
  - Trigger automations
  - Room-aware commands ("turn on the lights" → current room)

**SHOULD (Priority 2):**
- Direct smart device protocols (if HA unavailable):
  - Kasa/TP-Link
  - Zigbee/Z-Wave via USB stick
  - MQTT broker support

#### 3.3.2 Calendar & Scheduling
**MUST:**
- Google Calendar integration (read/write)
- Daily summary ("What's on my calendar?")
- Create/update events via voice
- Reminder notifications

**SHOULD:**
- Support multiple calendars
- CalDAV support (Nextcloud, etc.)
- Integration with task managers

#### 3.3.3 CRM (Contact Management)
**MUST:**
- Monica CRM integration (when deployed)
- Google Contacts sync
- Query contact information
- Log interactions with contacts

**SHOULD:**
- Track "last contacted" metrics
- Remind user to reach out to contacts
- Integration with n8n workflows

#### 3.3.4 Communication
**MUST:**
- Mattermost integration (notifications, chat interface)
- Send messages to Mattermost channels
- Receive notifications from Mattermost

**SHOULD:**
- Support two-way conversations via Mattermost
- Support @ mentions for bot interaction
- Integration with other chat platforms (Slack, Discord)

### 3.4 Admin Interface

#### 3.4.1 Web UI Requirements
**MUST:**
- Configuration management (integrations, models, endpoints)
- Real-time status dashboard (connected clients, system health)
- Conversation history viewer
- Memory browser/editor
- Integration testing interface
- System logs viewer

**SHOULD:**
- Mobile-responsive design
- Dark mode
- User authentication/authorization
- Multi-language support
- Plugin marketplace browser

#### 3.4.2 Configuration Management
**MUST:**
- Environment variable support
- YAML/JSON configuration files
- Hot-reload configuration changes (where safe)
- Validation of configuration on startup
- Sensible defaults for all settings

---

## 4. Non-Functional Requirements

### 4.1 Performance
- **Response Time:** <200ms for local queries, <2s for network/cloud
- **Audio Latency:** <100ms end-to-end (mic → speaker)
- **Concurrent Clients:** Support 5+ simultaneous connections
- **Memory Usage:** <2GB RAM idle, <4GB under load
- **CPU Usage:** <10% idle on 6-core system

### 4.2 Reliability
- **Uptime:** 99.9% (excluding planned maintenance)
- **Error Recovery:** Auto-restart on crash, graceful degradation
- **Data Integrity:** No conversation data loss on server restart
- **Network Resilience:** Clients auto-reconnect on network interruption

### 4.3 Security
- **Authentication:** Optional user authentication for admin UI
- **Encryption:** HTTPS/WSS for all network communication
- **Privacy:** No telemetry or analytics by default
- **Isolation:** Sandboxed execution for plugins/integrations

### 4.4 Scalability
- **Horizontal:** Support load balancing across multiple Rosie servers (future)
- **Vertical:** Efficient resource usage to run on modest hardware
- **Storage:** Graceful handling of large conversation histories (>10k messages)

### 4.5 Maintainability
- **Code Quality:** Type hints, linting (Ruff/Black), unit tests
- **Documentation:** API docs, architecture diagrams, inline comments
- **Logging:** Structured logs (JSON), configurable log levels
- **Monitoring:** Prometheus metrics export, health checks

### 4.6 Portability
- **OS Support:** Linux (primary), macOS, Windows (via Docker)
- **Architecture:** x86_64, ARM64 (Raspberry Pi)
- **Deployment:** Docker Compose (primary), Kubernetes (future), bare metal

---

## 5. Hardware Requirements

### 5.1 Minimum (Rosie Server)
- **CPU:** 4 cores @ 2GHz+ (x86_64 or ARM64)
- **RAM:** 8GB
- **Storage:** 20GB SSD
- **Network:** 100Mbps Ethernet or WiFi 5
- **OS:** Ubuntu 22.04+ or Debian 11+

### 5.2 Recommended (Rosie Server)
- **CPU:** 6+ cores @ 3GHz+
- **RAM:** 16GB+
- **Storage:** 50GB+ NVMe SSD
- **Network:** 1Gbps Ethernet
- **GPU:** Optional (AMD/NVIDIA for faster inference)

### 5.3 Optional GPU Server (Network AI)
- **GPU:** NVIDIA RTX 3060 (12GB) or better
- **CPU:** 4+ cores
- **RAM:** 16GB+
- **Network:** 1Gbps Ethernet (low latency to Rosie)

### 5.4 Client Endpoints
- **Echo Show Gen 1:** 1GB RAM, MediaTek MT8163 (supported)
- **Raspberry Pi 4/5:** 4GB+ RAM recommended
- **Any modern tablet/phone:** Web browser with WebRTC support

---

## 6. Technology Stack

### 6.1 Backend (Rosie Server)
- **Framework:** Python 3.11+ with FastAPI
- **LLM:** Ollama (local inference)
- **STT:** Faster-Whisper (OpenAI Whisper optimized)
- **TTS:** Piper TTS
- **Memory/RAG:** ChromaDB (vector database) + SQLite (metadata)
- **Real-time:** WebSocket (Socket.IO or native WebSocket)
- **Audio:** WebRTC for low-latency streaming
- **Task Queue:** Celery or asyncio (for background jobs)

### 6.2 Frontend (Admin UI)
- **Framework:** React 18+ or Vue 3 (TBD based on developer preference)
- **Styling:** TailwindCSS
- **State Management:** Context API or Pinia
- **Build Tool:** Vite
- **Type Safety:** TypeScript

### 6.3 Client Endpoints
- **Web Client:** HTML/CSS/JS with WebRTC
- **Echo Show:** Android WebView (Chromium) in kiosk mode
- **Raspberry Pi:** Minimal Chromium or Firefox kiosk

### 6.4 Infrastructure
- **Containerization:** Docker + Docker Compose
- **Reverse Proxy:** Caddy or Nginx
- **Process Management:** systemd or Docker restart policies
- **Monitoring:** Prometheus + Grafana (optional)

### 6.5 Development Tools
- **Linting:** Ruff (Python), ESLint (JS/TS)
- **Formatting:** Black (Python), Prettier (JS/TS)
- **Testing:** pytest (Python), Vitest (JS/TS)
- **CI/CD:** GitHub Actions
- **Documentation:** MkDocs or Docusaurus

---

## 7. Development Phases

### Phase 1: Core Voice Loop (MVP)
**Goal:** Working voice interaction end-to-end

**Deliverables:**
- FastAPI backend with WebSocket support
- Faster-Whisper STT integration
- Ollama LLM client (local 3B model)
- Piper TTS integration
- Simple web client for testing
- Docker Compose deployment

**Timeline:** 1-2 weeks

### Phase 2: Memory & Context
**Goal:** Conversations persist and recall past context

**Deliverables:**
- ChromaDB integration
- Conversation storage and retrieval
- RAG for semantic search
- Memory management UI

**Timeline:** 1 week

### Phase 3: First Integrations
**Goal:** Prove integration pattern works

**Deliverables:**
- Home Assistant integration
- Google Calendar integration
- Admin UI v1 (configuration, status)

**Timeline:** 1-2 weeks

### Phase 4: Multi-Client Support
**Goal:** Support multiple endpoints simultaneously

**Deliverables:**
- Room/client registration
- Context-aware routing
- Echo Show client guide
- Raspberry Pi client guide

**Timeline:** 1 week

### Phase 5: Advanced Features
**Goal:** Production-ready system

**Deliverables:**
- Network AI server routing (Ollama on GPU)
- Cloud API support (Gemini, OpenAI)
- Monica CRM + Mattermost integration
- Plugin architecture
- Full admin UI (logs, metrics, memory browser)

**Timeline:** 2-3 weeks

---

## 8. Out of Scope (v0.1)

**Explicitly NOT included in initial release:**
- Multi-user authentication (single-user or household assumed)
- Mobile apps (iOS/Android native)
- Video processing / computer vision
- Multi-language support (English only initially)
- Commercial integrations (Spotify, Netflix, etc.)
- Voice biometrics / speaker identification
- Kubernetes deployment
- High-availability / clustering

These may be added in future versions based on community feedback.

---

## 9. Open Questions & Decisions Needed

1. **Wake word engine:** Porcupine (proprietary but good) vs OpenWakeWord (open but newer)?
2. **Frontend framework:** React vs Vue? (Developer preference)
3. **Reverse proxy:** Caddy (simpler) vs Nginx (more flexible)?
4. **License:** MIT (permissive) vs Apache 2.0 (patent protection)?
5. **Plugin architecture:** Python-based vs language-agnostic (gRPC)?

---

## 10. Success Metrics

**Alpha Release (v0.1):**
- 10+ active deployments by community members
- <5 critical bugs reported per week
- <30 minute deployment time from clone to working
- Positive feedback on r/selfhosted or similar communities

**Beta Release (v0.5):**
- 100+ active deployments
- 5+ community-contributed integrations
- <1 critical bug per month
- Documentation coverage >80%

**v1.0 Release:**
- 500+ active deployments
- Featured on Awesome Self-Hosted list
- 20+ contributors
- Production-ready stability (99.9% uptime)
