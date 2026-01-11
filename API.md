# Rosie API Reference

**Version:** 0.1.0  
**Last Updated:** January 2026  
**Base URL:** `http://rosie.local:8000`

---

## 1. Overview

Rosie exposes a REST API for configuration and control, plus WebSocket/WebRTC endpoints for real-time voice interaction.

**Authentication:** None (v0.1) - Assumes trusted network  
**Content-Type:** `application/json`  
**Rate Limiting:** 100 requests/minute per IP (future)

---

## 2. REST API

### 2.1 Health & Status

#### `GET /health`

**Description:** Basic health check (always responds if server alive)

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2026-01-11T12:34:56Z"
}
```

**Status Codes:**
- `200 OK` - Server is healthy

---

#### `GET /api/v1/health/detailed`

**Description:** Detailed health check (tests all components)

**Response:**
```json
{
  "status": "healthy",
  "components": {
    "stt": {
      "status": "ok",
      "latency_ms": 234
    },
    "llm": {
      "status": "ok",
      "model": "llama3.2:3b",
      "latency_ms": 567
    },
    "tts": {
      "status": "ok",
      "latency_ms": 123
    },
    "memory": {
      "status": "ok",
      "conversations": 42,
      "embeddings": 1537
    }
  },
  "timestamp": "2026-01-11T12:34:56Z"
}
```

**Status Codes:**
- `200 OK` - All components healthy
- `503 Service Unavailable` - One or more components degraded

---

#### `GET /api/v1/status`

**Description:** System status and metrics

**Response:**
```json
{
  "version": "0.1.0",
  "uptime_seconds": 3672,
  "connected_clients": 3,
  "clients": [
    {
      "room_id": "office",
      "connected_at": "2026-01-11T11:00:00Z",
      "last_activity": "2026-01-11T12:30:00Z"
    },
    {
      "room_id": "kitchen",
      "connected_at": "2026-01-11T10:15:00Z",
      "last_activity": "2026-01-11T12:34:56Z"
    }
  ],
  "resource_usage": {
    "cpu_percent": 23.4,
    "memory_mb": 1842,
    "disk_mb": 12450
  },
  "requests_per_minute": 12.3
}
```

**Status Codes:**
- `200 OK`

---

### 2.2 Configuration

#### `GET /api/v1/config`

**Description:** Get current configuration

**Response:**
```json
{
  "stt": {
    "engine": "faster-whisper",
    "model": "base.en",
    "device": "cuda"
  },
  "llm": {
    "local": {
      "provider": "ollama",
      "model": "llama3.2:3b",
      "host": "http://localhost:11434"
    },
    "network": {
      "enabled": true,
      "provider": "ollama",
      "host": "http://sofia.local:11434",
      "model": "llama3.3:70b"
    },
    "cloud": {
      "enabled": false,
      "provider": "gemini",
      "model": "gemini-2.0-flash"
    }
  },
  "tts": {
    "engine": "piper",
    "voice": "en_US-lessac-medium"
  },
  "memory": {
    "max_conversations": 10000,
    "retention_days": 365
  }
}
```

**Status Codes:**
- `200 OK`

---

#### `POST /api/v1/config`

**Description:** Update configuration

**Request Body:**
```json
{
  "llm": {
    "local": {
      "model": "phi3:3.8b"
    }
  },
  "tts": {
    "voice": "en_US-amy-medium"
  }
}
```

**Response:**
```json
{
  "success": true,
  "message": "Configuration updated",
  "restarted_services": ["llm", "tts"]
}
```

**Status Codes:**
- `200 OK` - Configuration updated
- `400 Bad Request` - Invalid configuration
- `500 Internal Server Error` - Update failed

---

### 2.3 Memory & Conversations

#### `GET /api/v1/memory/conversations`

**Description:** List recent conversations

**Query Parameters:**
- `limit` (int, default: 20) - Number of conversations to return
- `room_id` (string, optional) - Filter by room
- `offset` (int, default: 0) - Pagination offset

**Response:**
```json
{
  "conversations": [
    {
      "id": "conv_abc123",
      "room_id": "office",
      "started_at": "2026-01-11T10:00:00Z",
      "ended_at": "2026-01-11T10:05:32Z",
      "message_count": 8,
      "summary": "Discussion about weather and calendar"
    },
    {
      "id": "conv_def456",
      "room_id": "kitchen",
      "started_at": "2026-01-11T09:00:00Z",
      "ended_at": "2026-01-11T09:02:15Z",
      "message_count": 4,
      "summary": "Turned on lights"
    }
  ],
  "total": 42,
  "limit": 20,
  "offset": 0
}
```

**Status Codes:**
- `200 OK`

---

#### `GET /api/v1/memory/conversations/:id`

**Description:** Get conversation details

**Response:**
```json
{
  "id": "conv_abc123",
  "room_id": "office",
  "started_at": "2026-01-11T10:00:00Z",
  "ended_at": "2026-01-11T10:05:32Z",
  "messages": [
    {
      "id": "msg_001",
      "timestamp": "2026-01-11T10:00:12Z",
      "speaker": "user",
      "text": "What's the weather today?"
    },
    {
      "id": "msg_002",
      "timestamp": "2026-01-11T10:00:14Z",
      "speaker": "rosie",
      "text": "It's 72°F and sunny in Ellenton, Florida."
    }
  ]
}
```

**Status Codes:**
- `200 OK`
- `404 Not Found` - Conversation not found

---

#### `POST /api/v1/memory/search`

**Description:** Semantic search across conversations

**Request Body:**
```json
{
  "query": "What did we discuss about the HOA meeting?",
  "room_id": "office",
  "limit": 5
}
```

**Response:**
```json
{
  "results": [
    {
      "conversation_id": "conv_xyz789",
      "message_id": "msg_042",
      "score": 0.87,
      "timestamp": "2026-01-10T14:30:00Z",
      "speaker": "user",
      "text": "Can you remind me what we decided at the HOA meeting yesterday?",
      "context": "... surrounding messages ..."
    }
  ]
}
```

**Status Codes:**
- `200 OK`
- `400 Bad Request` - Invalid query

---

#### `DELETE /api/v1/memory/conversations/:id`

**Description:** Delete a conversation

**Response:**
```json
{
  "success": true,
  "message": "Conversation deleted"
}
```

**Status Codes:**
- `200 OK`
- `404 Not Found` - Conversation not found

---

#### `POST /api/v1/memory/export`

**Description:** Export all conversation data

**Response:**
```json
{
  "export_url": "/api/v1/memory/downloads/export_20260111.json",
  "expires_at": "2026-01-11T13:00:00Z",
  "size_bytes": 1234567
}
```

**Status Codes:**
- `200 OK`
- `500 Internal Server Error` - Export failed

---

### 2.4 Integrations

#### `GET /api/v1/integrations`

**Description:** List available integrations

**Response:**
```json
{
  "integrations": [
    {
      "id": "home_assistant",
      "name": "Home Assistant",
      "version": "1.0.0",
      "status": "configured",
      "healthy": true
    },
    {
      "id": "google_calendar",
      "name": "Google Calendar",
      "version": "1.0.0",
      "status": "not_configured",
      "healthy": null
    }
  ]
}
```

**Status Codes:**
- `200 OK`

---

#### `POST /api/v1/integrations/:id/configure`

**Description:** Configure an integration

**Request Body (example for Home Assistant):**
```json
{
  "base_url": "http://homeassistant.local:8123",
  "api_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

**Response:**
```json
{
  "success": true,
  "message": "Home Assistant configured successfully",
  "status": "configured",
  "healthy": true
}
```

**Status Codes:**
- `200 OK` - Configuration successful
- `400 Bad Request` - Invalid configuration
- `503 Service Unavailable` - Integration unreachable

---

#### `POST /api/v1/integrations/:id/test`

**Description:** Test integration connection

**Response:**
```json
{
  "healthy": true,
  "message": "Connected successfully",
  "latency_ms": 123
}
```

**Status Codes:**
- `200 OK` - Test passed
- `503 Service Unavailable` - Test failed

---

### 2.5 Logs

#### `GET /api/v1/logs`

**Description:** Stream server logs (SSE)

**Query Parameters:**
- `level` (string, default: "INFO") - Minimum log level (DEBUG, INFO, WARN, ERROR)
- `component` (string, optional) - Filter by component (stt, llm, tts, memory)

**Response:** Server-Sent Events stream
```
data: {"timestamp": "2026-01-11T12:34:56Z", "level": "INFO", "component": "stt", "message": "Transcription complete"}

data: {"timestamp": "2026-01-11T12:34:57Z", "level": "INFO", "component": "llm", "message": "Generated response"}
```

**Status Codes:**
- `200 OK`

---

## 3. WebSocket API

### 3.1 Real-Time Voice Stream

#### `WS /api/v1/stream`

**Description:** Bi-directional audio/control stream

**Connection:**
```javascript
const ws = new WebSocket('ws://rosie.local:8000/api/v1/stream');

ws.onopen = () => {
  // Register client
  ws.send(JSON.stringify({
    type: 'register',
    room_id: 'office',
    capabilities: ['audio_input', 'audio_output', 'display']
  }));
};
```

---

### 3.2 Message Types

#### Client → Server: Register

```json
{
  "type": "register",
  "room_id": "office",
  "capabilities": ["audio_input", "audio_output", "display"]
}
```

**Response:**
```json
{
  "type": "registered",
  "session_id": "sess_abc123",
  "server_version": "0.1.0"
}
```

---

#### Client → Server: Audio Chunk

```json
{
  "type": "audio",
  "data": "<base64 encoded audio>",
  "format": "pcm_16khz_mono",
  "sequence": 42
}
```

---

#### Client → Server: Manual Input

```json
{
  "type": "text_input",
  "text": "What's the weather?"
}
```

---

#### Server → Client: Transcription

```json
{
  "type": "transcription",
  "text": "What's the weather?",
  "partial": false,
  "timestamp": "2026-01-11T12:34:56Z"
}
```

---

#### Server → Client: Response Chunk

```json
{
  "type": "response",
  "text": "It's 72°F and sunny",
  "partial": true,
  "timestamp": "2026-01-11T12:34:57Z"
}
```

---

#### Server → Client: Audio Chunk (TTS)

```json
{
  "type": "audio",
  "data": "<base64 encoded audio>",
  "format": "wav_22khz_mono",
  "sequence": 1
}
```

---

#### Server → Client: Status Update

```json
{
  "type": "status",
  "state": "listening",
  "message": "Listening..."
}
```

**States:**
- `idle` - Not actively processing
- `listening` - Capturing audio (VAD active)
- `transcribing` - Processing speech
- `thinking` - Generating response
- `speaking` - Playing TTS audio

---

#### Server → Client: Error

```json
{
  "type": "error",
  "code": "STT_FAILED",
  "message": "Speech recognition failed",
  "recoverable": true
}
```

**Error Codes:**
- `STT_FAILED` - Speech-to-text error
- `LLM_FAILED` - LLM generation error
- `TTS_FAILED` - Text-to-speech error
- `INTEGRATION_FAILED` - Integration command failed
- `INVALID_MESSAGE` - Malformed client message

---

## 4. WebRTC API (Future)

**Note:** WebRTC support is planned for lower-latency audio streaming. Currently using WebSocket for simplicity.

---

## 5. Error Responses

**Standard Error Format:**
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: 'room_id'",
    "details": {
      "field": "room_id",
      "expected": "string"
    }
  }
}
```

**Common HTTP Status Codes:**
- `200 OK` - Success
- `400 Bad Request` - Invalid request
- `401 Unauthorized` - Authentication required (future)
- `404 Not Found` - Resource not found
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Service temporarily unavailable

---

## 6. Rate Limiting (Future)

**Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1641910800
```

**When Exceeded:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retry_after_seconds": 42
  }
}
```

---

## 7. Versioning

**Current Version:** v1  
**Base Path:** `/api/v1/`

**Future versions:** `/api/v2/`, etc.

**Deprecation:** v1 will be supported for at least 12 months after v2 release.

---

## 8. Client Libraries

**Official:**
- Python: `rosie-client` (coming soon)
- JavaScript/TypeScript: `@rosie/client` (coming soon)

**Community:**
- (Contributions welcome!)

---

## 9. OpenAPI Specification

**Interactive Docs:** `http://rosie.local:8000/docs` (Swagger UI)

**OpenAPI JSON:** `http://rosie.local:8000/openapi.json`

---

## 10. Example Usage

### 10.1 Python

```python
import asyncio
import websockets
import json
import base64

async def voice_loop():
    uri = "ws://rosie.local:8000/api/v1/stream"
    
    async with websockets.connect(uri) as ws:
        # Register
        await ws.send(json.dumps({
            "type": "register",
            "room_id": "office"
        }))
        
        response = await ws.recv()
        print(f"Registered: {response}")
        
        # Send audio (example)
        with open("audio.pcm", "rb") as f:
            audio_data = f.read()
            await ws.send(json.dumps({
                "type": "audio",
                "data": base64.b64encode(audio_data).decode(),
                "format": "pcm_16khz_mono"
            }))
        
        # Receive responses
        async for message in ws:
            data = json.loads(message)
            if data["type"] == "transcription":
                print(f"Transcription: {data['text']}")
            elif data["type"] == "response":
                print(f"Response: {data['text']}")
            elif data["type"] == "audio":
                # Play TTS audio
                audio = base64.b64decode(data["data"])
                # ... play audio ...

asyncio.run(voice_loop())
```

---

### 10.2 JavaScript

```javascript
const ws = new WebSocket('ws://rosie.local:8000/api/v1/stream');

ws.onopen = () => {
  // Register
  ws.send(JSON.stringify({
    type: 'register',
    room_id: 'office'
  }));
  
  // Start capturing audio
  navigator.mediaDevices.getUserMedia({ audio: true })
    .then(stream => {
      const audioContext = new AudioContext();
      const source = audioContext.createMediaStreamSource(stream);
      const processor = audioContext.createScriptProcessor(4096, 1, 1);
      
      processor.onaudioprocess = (e) => {
        const audioData = e.inputBuffer.getChannelData(0);
        // Convert to PCM and send
        ws.send(JSON.stringify({
          type: 'audio',
          data: btoa(String.fromCharCode(...new Uint8Array(audioData.buffer))),
          format: 'pcm_16khz_mono'
        }));
      };
      
      source.connect(processor);
      processor.connect(audioContext.destination);
    });
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  if (data.type === 'transcription') {
    console.log('Transcription:', data.text);
  } else if (data.type === 'response') {
    console.log('Response:', data.text);
  } else if (data.type === 'audio') {
    // Play TTS audio
    const audioData = atob(data.data);
    // ... play audio ...
  }
};
```

---

## End of API Reference

**For more examples, see:**
- `/examples` directory in repo
- Integration guides in `/docs`
- Community contributions
