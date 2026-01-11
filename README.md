# R.O.S.I.E.

**Resident Operator for Smart Infrastructure & Execution**

A privacy-first, local-only voice assistant with modular architecture. Run your own AI assistant on modest hardware with zero cloud dependencies.

## 🎯 Philosophy

- **Privacy-First:** All processing happens locally on your hardware
- **Modular:** Swap LLMs, STT/TTS engines, and integrations easily  
- **Network-Native:** Central server, multiple endpoints (Echo Show, Pi, tablets)
- **Resource-Efficient:** Runs on consumer hardware (no datacenter required)
- **Open Ecosystem:** Bring your own endpoints, models, and integrations

## ✨ Features

- 🎙️ Voice interaction with wake word detection
- 🧠 Local LLM inference (Ollama integration)
- 🏠 Home automation (Home Assistant, smart devices)
- 📅 Calendar & scheduling
- 💾 Conversational memory with RAG
- 🌐 Web-based admin interface
- 🔌 Extensible plugin architecture

## 🚀 Quick Start
```bash
# Clone the repo
git clone https://github.com/screscenti/rosie-io.git
cd rosie-io

# Start with Docker Compose (minimal setup)
docker-compose -f examples/docker-compose.minimal.yml up

# Access admin UI at http://localhost:8000
```

## 📋 Requirements

**Minimum:**
- CPU: 4 cores (2GHz+)
- RAM: 8GB
- Storage: 20GB
- OS: Linux (Ubuntu 22.04+ recommended)

**Recommended:**
- CPU: 6+ cores
- RAM: 16GB
- GPU: Optional but helpful (AMD/NVIDIA)

## 🛠️ Supported Endpoints

- **Jailbroken Echo Show** (1st gen) - Best UX
- **Raspberry Pi** (4/5 with display)
- **Any tablet/PC** (web browser)
- **Mobile devices** (coming soon)

## 📖 Documentation

- [Architecture Overview](docs/ARCHITECTURE.md)
- [Hardware Requirements](docs/HARDWARE.md)
- [Development Guide](docs/DEVELOPMENT.md)
- [API Reference](docs/API.md)

## 🤝 Contributing

This project is in active development. See [DEVELOPMENT.md](docs/DEVELOPMENT.md) for contribution guidelines.

## 📄 License

MIT License - see [LICENSE](LICENSE) for details

## 🙏 Acknowledgments

Inspired by the desire for truly private, powerful voice assistants that respect user autonomy.
