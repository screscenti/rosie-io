# Rosie Development Guide

**Version:** 0.1.0  
**Last Updated:** January 2026

---

## 1. Getting Started

### 1.1 Prerequisites

**Required:**
- **Git** (for cloning repo)
- **Docker** + **Docker Compose** (for containerized deployment)
- **Python 3.11+** (for local development)
- **Node.js 18+** (for web UI development)

**Optional:**
- **NVIDIA Docker Runtime** (for GPU support)
- **ROCm** (for AMD GPU support)

---

### 1.2 Quick Start (Docker Compose)

**Clone the repo:**
```bash
git clone https://github.com/screscenti/rosie-io.git
cd rosie-io
```

**Start minimal stack:**
```bash
docker-compose -f examples/docker-compose.minimal.yml up
```

**Access:**
- Admin UI: http://localhost:3000
- API: http://localhost:8000
- API Docs: http://localhost:8000/docs

---

## 2. Development Environment Setup

### 2.1 Backend (Python)

**Install dependencies:**
```bash
cd server
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Linting, testing tools
```

**Run backend locally:**
```bash
# Set environment variables
export OLLAMA_HOST=http://localhost:11434
export LOG_LEVEL=DEBUG

# Start server
python main.py
```

**Run tests:**
```bash
pytest tests/ -v
```

**Linting:**
```bash
ruff check server/
black server/ --check
```

---

### 2.2 Frontend (Web UI)

**Install dependencies:**
```bash
cd web-ui
npm install
```

**Run dev server:**
```bash
npm run dev
# Accessible at http://localhost:5173
```

**Build for production:**
```bash
npm run build
# Output in dist/
```

**Linting:**
```bash
npm run lint
npm run format
```

---

### 2.3 Docker Development

**Build images locally:**
```bash
docker-compose build
```

**Run with hot-reload (backend):**
```bash
docker-compose -f docker-compose.dev.yml up
# Mounts ./server as volume for live code changes
```

---

## 3. Project Structure

```
rosie-io/
├── server/                     # Python backend
│   ├── main.py                 # FastAPI entry point
│   ├── config.py               # Configuration management
│   ├── api/                    # API routes
│   │   ├── __init__.py
│   │   ├── health.py           # Health check endpoints
│   │   ├── stream.py           # WebSocket/WebRTC
│   │   ├── memory.py           # Memory/RAG endpoints
│   │   └── admin.py            # Admin/config endpoints
│   ├── services/               # Core services
│   │   ├── __init__.py
│   │   ├── stt.py              # Speech-to-text (Faster-Whisper)
│   │   ├── llm.py              # LLM routing (Ollama client)
│   │   ├── tts.py              # Text-to-speech (Piper)
│   │   └── memory.py           # ChromaDB + SQLite
│   ├── integrations/           # Third-party integrations
│   │   ├── __init__.py
│   │   ├── base.py             # Integration plugin base class
│   │   ├── home_assistant.py   # Home Assistant
│   │   ├── calendar.py         # Google Calendar
│   │   └── mattermost.py       # Mattermost
│   ├── models/                 # Data models (Pydantic)
│   │   ├── __init__.py
│   │   ├── conversation.py
│   │   ├── config.py
│   │   └── integration.py
│   ├── utils/                  # Utility functions
│   │   ├── __init__.py
│   │   ├── logger.py
│   │   └── audio.py
│   ├── tests/                  # Unit tests
│   │   ├── test_stt.py
│   │   ├── test_llm.py
│   │   └── test_memory.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── requirements-dev.txt
├── web-ui/                     # Web admin interface
│   ├── src/
│   │   ├── App.jsx             # Root component
│   │   ├── main.jsx            # Entry point
│   │   ├── components/         # React components
│   │   ├── pages/              # Page components
│   │   ├── hooks/              # Custom React hooks
│   │   ├── api/                # API client
│   │   └── utils/              # Utility functions
│   ├── public/
│   ├── package.json
│   ├── vite.config.js
│   └── Dockerfile
├── endpoints/                  # Client endpoint guides
│   ├── echo-show/
│   │   ├── README.md           # Jailbreak & setup guide
│   │   └── scripts/
│   ├── raspberry-pi/
│   │   ├── README.md           # Pi kiosk setup
│   │   └── scripts/
│   └── generic-web/
│       └── index.html          # Standalone web client
├── examples/
│   ├── docker-compose.minimal.yml
│   ├── docker-compose.full.yml
│   └── integrations/
│       ├── home-assistant-config.yaml
│       └── google-calendar-setup.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── REQUIREMENTS.md
│   ├── HARDWARE.md
│   ├── DEVELOPMENT.md          # This file
│   └── API.md
├── .github/
│   └── workflows/
│       ├── ci.yml              # CI/CD pipeline
│       └── release.yml
├── docker-compose.yml          # Production stack
├── .gitignore
├── LICENSE
└── README.md
```

---

## 4. Core Components

### 4.1 Backend Services

#### 4.1.1 STT Service (server/services/stt.py)

**Purpose:** Transcribe audio to text

**Key Functions:**
```python
async def transcribe_stream(audio_stream: AsyncIterator[bytes]) -> AsyncIterator[str]:
    """
    Transcribe audio stream in real-time.
    
    Args:
        audio_stream: Async iterator of audio bytes (PCM/Opus)
    
    Yields:
        Partial transcription strings
    """
    pass

async def transcribe_file(audio_path: str) -> str:
    """
    Transcribe audio file (for testing/batch processing).
    
    Args:
        audio_path: Path to audio file
    
    Returns:
        Full transcription
    """
    pass
```

**Dependencies:**
- `faster-whisper` (CTranslate2 optimized Whisper)
- `torch` (for model loading)

---

#### 4.1.2 LLM Service (server/services/llm.py)

**Purpose:** Route queries to appropriate LLM tier

**Key Functions:**
```python
async def generate(
    prompt: str,
    context: dict,
    stream: bool = True
) -> AsyncIterator[str]:
    """
    Generate LLM response with tiered routing.
    
    Args:
        prompt: User query
        context: Conversation context, user preferences, etc.
        stream: Whether to stream response word-by-word
    
    Yields:
        Response chunks
    """
    pass

def classify_complexity(prompt: str) -> str:
    """
    Classify query complexity (simple/medium/complex).
    
    Args:
        prompt: User query
    
    Returns:
        "simple" | "medium" | "complex"
    """
    pass

def route_query(complexity: str, config: dict) -> LLMTier:
    """
    Determine which LLM tier to use.
    
    Args:
        complexity: Query complexity
        config: User preferences, available resources
    
    Returns:
        LLMTier.LOCAL | LLMTier.NETWORK | LLMTier.CLOUD
    """
    pass
```

**Dependencies:**
- `ollama` (Python client for Ollama API)
- `openai` (for cloud API compatibility)

---

#### 4.1.3 TTS Service (server/services/tts.py)

**Purpose:** Synthesize speech from text

**Key Functions:**
```python
async def synthesize(text: str, voice: str = "default") -> bytes:
    """
    Synthesize speech audio.
    
    Args:
        text: Text to speak
        voice: Voice ID (e.g., "en_US-lessac-medium")
    
    Returns:
        Audio bytes (WAV format)
    """
    pass

async def synthesize_stream(text: str) -> AsyncIterator[bytes]:
    """
    Stream audio as it's synthesized (for faster playback start).
    
    Args:
        text: Text to speak
    
    Yields:
        Audio chunks
    """
    pass
```

**Dependencies:**
- `piper-tts` (fast local TTS)

---

#### 4.1.4 Memory Service (server/services/memory.py)

**Purpose:** Store and retrieve conversation history

**Key Functions:**
```python
async def store_message(
    room_id: str,
    speaker: str,
    text: str,
    metadata: dict = {}
):
    """
    Store a conversation message.
    
    Args:
        room_id: Which endpoint/room
        speaker: "user" or "rosie"
        text: Message text
        metadata: Extra data (intent, tools used, etc.)
    """
    pass

async def search(
    query: str,
    room_id: str = None,
    k: int = 5
) -> List[dict]:
    """
    Semantic search across conversation history.
    
    Args:
        query: Search query
        room_id: Optional filter by room
        k: Number of results
    
    Returns:
        List of relevant messages with scores
    """
    pass

async def get_recent(room_id: str, n: int = 10) -> List[dict]:
    """
    Get last N messages from a room.
    
    Args:
        room_id: Room identifier
        n: Number of messages
    
    Returns:
        List of messages (newest first)
    """
    pass
```

**Dependencies:**
- `chromadb` (vector database)
- `sqlite3` (metadata storage)
- `sentence-transformers` (for embeddings)

---

### 4.2 API Routes

#### 4.2.1 WebSocket Stream (api/stream.py)

**Purpose:** Real-time client communication

**Endpoints:**
```
WS /api/v1/stream
```

**Message Types:**
```json
// Client → Server: Audio chunk
{
  "type": "audio",
  "data": "<base64 audio>",
  "room_id": "office"
}

// Server → Client: Transcription
{
  "type": "transcription",
  "text": "What's the weather?",
  "partial": false
}

// Server → Client: Response chunk
{
  "type": "response",
  "text": "It's 72°F and sunny",
  "partial": true
}

// Server → Client: Audio chunk (TTS)
{
  "type": "audio",
  "data": "<base64 audio>"
}
```

---

#### 4.2.2 Memory API (api/memory.py)

**Endpoints:**
```
GET  /api/v1/memory/conversations       # List recent conversations
GET  /api/v1/memory/conversations/:id   # Get conversation details
POST /api/v1/memory/search              # Semantic search
DELETE /api/v1/memory/conversations/:id # Delete conversation
POST /api/v1/memory/export              # Export all data (JSON)
```

---

#### 4.2.3 Admin API (api/admin.py)

**Endpoints:**
```
GET  /api/v1/config             # Get current configuration
POST /api/v1/config             # Update configuration
GET  /api/v1/status             # System status (connected clients, resource usage)
GET  /api/v1/integrations       # List available integrations
POST /api/v1/integrations/:id   # Configure integration
GET  /api/v1/logs               # Stream server logs
```

---

## 5. Integration Development

### 5.1 Creating a New Integration

**1. Create integration file:**
```python
# server/integrations/my_integration.py

from .base import IntegrationPlugin
from typing import Dict, Any

class MyIntegration(IntegrationPlugin):
    """
    Integration with MyService.
    """
    
    name = "my_integration"
    version = "1.0.0"
    
    async def initialize(self, config: Dict[str, Any]):
        """
        Set up integration with API keys, etc.
        
        Args:
            config: Dict with "api_key", "base_url", etc.
        """
        self.api_key = config["api_key"]
        self.base_url = config.get("base_url", "https://api.myservice.com")
        # Test connection
        await self._test_connection()
    
    async def handle_command(self, intent: str, entities: Dict) -> Dict:
        """
        Execute command.
        
        Args:
            intent: Command intent (e.g., "do_something")
            entities: Extracted entities (e.g., {"item": "lights"})
        
        Returns:
            Result dict: {"success": bool, "message": str, "data": Any}
        """
        if intent == "do_something":
            result = await self._api_call("/endpoint", data=entities)
            return {"success": True, "message": "Done!", "data": result}
        
        return {"success": False, "message": f"Unknown intent: {intent}"}
    
    async def get_status(self) -> Dict:
        """
        Return integration health/status.
        
        Returns:
            {"healthy": bool, "message": str}
        """
        try:
            await self._test_connection()
            return {"healthy": True, "message": "Connected"}
        except Exception as e:
            return {"healthy": False, "message": str(e)}
    
    def get_intents(self) -> List[str]:
        """
        Return list of supported intents.
        """
        return ["do_something", "do_another_thing"]
```

**2. Register integration:**
```python
# server/integrations/__init__.py

from .my_integration import MyIntegration

AVAILABLE_INTEGRATIONS = {
    "my_integration": MyIntegration,
    # ... other integrations
}
```

**3. Test integration:**
```python
# server/tests/test_my_integration.py

import pytest
from server.integrations.my_integration import MyIntegration

@pytest.mark.asyncio
async def test_my_integration():
    integration = MyIntegration()
    await integration.initialize({"api_key": "test_key"})
    
    result = await integration.handle_command(
        intent="do_something",
        entities={"item": "test"}
    )
    
    assert result["success"] == True
```

---

### 5.2 LLM Function Calling

**Define functions for LLM:**
```python
# In server/services/llm.py

AVAILABLE_FUNCTIONS = [
    {
        "name": "my_integration.do_something",
        "description": "Do something with MyService",
        "parameters": {
            "type": "object",
            "properties": {
                "item": {
                    "type": "string",
                    "description": "The item to act on"
                }
            },
            "required": ["item"]
        }
    }
]
```

**LLM calls function:**
```python
# User: "Do something with lights"
# LLM generates:
{
  "function": "my_integration.do_something",
  "arguments": {"item": "lights"}
}

# Execute function
integration = get_integration("my_integration")
result = await integration.handle_command(
    intent="do_something",
    entities={"item": "lights"}
)

# Feed result back to LLM for natural language response
response = await llm.generate(
    f"Function result: {result}. Generate natural response."
)
# "I've done something with the lights!"
```

---

## 6. Testing

### 6.1 Unit Tests

**Run all tests:**
```bash
pytest server/tests/ -v
```

**Run specific test:**
```bash
pytest server/tests/test_stt.py::test_transcribe -v
```

**Test coverage:**
```bash
pytest --cov=server --cov-report=html
# Open htmlcov/index.html
```

---

### 6.2 Integration Tests

**Test full voice loop:**
```bash
pytest server/tests/integration/test_voice_loop.py -v
```

**Test with real audio file:**
```bash
# server/tests/test_data/test_audio.wav
pytest server/tests/test_stt.py::test_transcribe_file -v
```

---

### 6.3 Manual Testing

**Use web client:**
1. Start server: `docker-compose up`
2. Open browser: `http://localhost:3000`
3. Click "Start Listening"
4. Speak into microphone
5. Verify transcription and response

**Use API directly:**
```bash
# Test health endpoint
curl http://localhost:8000/api/v1/health

# Test LLM (requires server running)
curl -X POST http://localhost:8000/api/v1/llm/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the capital of France?"}'
```

---

## 7. Code Style & Standards

### 7.1 Python

**Linting:** Ruff (fast, comprehensive)
```bash
ruff check server/
```

**Formatting:** Black
```bash
black server/
```

**Type Hints:** Required
```python
def process_audio(audio: bytes, sample_rate: int = 16000) -> str:
    """Process audio and return transcription."""
    pass
```

**Docstrings:** Google style
```python
def function(arg1: str, arg2: int) -> bool:
    """
    Short description.
    
    Longer description if needed.
    
    Args:
        arg1: Description of arg1
        arg2: Description of arg2
    
    Returns:
        Description of return value
    
    Raises:
        ValueError: When something is wrong
    """
    pass
```

---

### 7.2 JavaScript/TypeScript

**Linting:** ESLint
```bash
npm run lint
```

**Formatting:** Prettier
```bash
npm run format
```

**Type Safety:** TypeScript (preferred) or JSDoc
```typescript
function processData(data: string[]): number {
  return data.length;
}
```

---

## 8. Git Workflow

### 8.1 Branching Strategy

**Branches:**
- `main` - Stable, production-ready code
- `develop` - Integration branch for features
- `feature/*` - Feature branches (e.g., `feature/home-assistant-integration`)
- `fix/*` - Bug fix branches

**Example:**
```bash
# Create feature branch
git checkout -b feature/add-calendar-integration

# Make changes, commit
git add .
git commit -m "feat: Add Google Calendar integration"

# Push and create PR
git push origin feature/add-calendar-integration
```

---

### 8.2 Commit Messages

**Format:** Conventional Commits
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, no logic change)
- `refactor`: Code refactor (no feature/fix)
- `test`: Add/update tests
- `chore`: Maintenance (dependencies, config)

**Examples:**
```
feat(stt): Add streaming transcription support

Implemented real-time transcription using Faster-Whisper's
streaming API. Reduces latency by ~500ms.

Closes #42

---

fix(memory): Fix ChromaDB connection timeout

Increased connection timeout from 5s to 30s to handle
slow network conditions.

Fixes #78
```

---

### 8.3 Pull Request Process

**1. Create PR:**
- Title: Clear, descriptive
- Description: What, why, how
- Link related issues

**2. CI Checks:**
- Linting passes
- Tests pass
- No merge conflicts

**3. Code Review:**
- At least 1 approval required
- Address feedback

**4. Merge:**
- Squash commits (keep history clean)
- Delete branch after merge

---

## 9. CI/CD Pipeline

### 9.1 GitHub Actions

**Workflow: `.github/workflows/ci.yml`**

**On every push/PR:**
1. Lint Python (Ruff)
2. Lint JS/TS (ESLint)
3. Run Python tests (pytest)
4. Run JS tests (Vitest)
5. Build Docker images
6. Check documentation links

**Workflow: `.github/workflows/release.yml`**

**On tag push (v*):**
1. Run full CI
2. Build and push Docker images to Docker Hub
3. Create GitHub release
4. Generate changelog

---

## 10. Release Process

**1. Version Bump:**
```bash
# Update version in:
# - server/config.py (VERSION = "0.2.0")
# - web-ui/package.json ("version": "0.2.0")
# - README.md

git add .
git commit -m "chore: Bump version to 0.2.0"
```

**2. Tag Release:**
```bash
git tag -a v0.2.0 -m "Release v0.2.0"
git push origin v0.2.0
```

**3. GitHub Actions builds and publishes**

**4. Update Documentation:**
- Update CHANGELOG.md
- Update docs with new features

---

## 11. Troubleshooting

### 11.1 Common Issues

**Problem:** Docker build fails with "No space left on device"

**Solution:**
```bash
# Clean up Docker
docker system prune -a
```

---

**Problem:** Ollama model not loading

**Solution:**
```bash
# Pull model manually
docker exec rosie-ollama ollama pull llama3.2:3b

# Check logs
docker logs rosie-ollama
```

---

**Problem:** ChromaDB connection errors

**Solution:**
```bash
# Check ChromaDB container
docker logs rosie-chromadb

# Restart ChromaDB
docker-compose restart chromadb
```

---

**Problem:** Client can't connect to server

**Solution:**
```bash
# Check firewall
sudo ufw allow 8000/tcp

# Check server logs
docker logs rosie-server

# Verify WebSocket endpoint
curl -i -N -H "Connection: Upgrade" \
  -H "Upgrade: websocket" \
  http://localhost:8000/api/v1/stream
```

---

## 12. Development Best Practices

**1. Write tests first (TDD)**
- Define expected behavior
- Write failing test
- Implement feature
- Test passes

**2. Keep functions small and focused**
- Single responsibility principle
- Easier to test and maintain

**3. Use type hints everywhere**
- Python: Type hints
- JS: TypeScript or JSDoc
- Catches bugs early

**4. Document non-obvious code**
- Why, not what
- Edge cases, assumptions

**5. Test error paths**
- Not just happy path
- Network failures, invalid input

**6. Profile before optimizing**
- Measure first
- Optimize bottlenecks, not assumptions

---

## 13. Resources

**Python:**
- FastAPI Docs: https://fastapi.tiangolo.com
- Pydantic: https://docs.pydantic.dev
- Pytest: https://docs.pytest.org

**AI/ML:**
- Ollama: https://ollama.ai/docs
- Faster-Whisper: https://github.com/guillaumekln/faster-whisper
- ChromaDB: https://docs.trychroma.com

**Frontend:**
- React: https://react.dev
- Vite: https://vitejs.dev
- TailwindCSS: https://tailwindcss.com

**Docker:**
- Docker Docs: https://docs.docker.com
- Docker Compose: https://docs.docker.com/compose

---

## End of Development Guide

**Ready to contribute? Check out:**
- Open issues: https://github.com/screscenti/rosie-io/issues
- Project roadmap: https://github.com/screscenti/rosie-io/projects
- Discord (if created): [link]
