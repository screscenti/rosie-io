# Rosie Hardware Requirements & Recommendations

**Version:** 0.1.0  
**Last Updated:** January 2026

---

## 1. Overview

Rosie is designed to run efficiently on consumer hardware. This document outlines minimum, recommended, and optimal configurations for both the central server and client endpoints.

**Key Principles:**
- **Accessible:** Run on modest hardware (no datacenter needed)
- **Scalable:** Start small, upgrade as needed
- **Flexible:** Support CPU-only or GPU-accelerated deployments

---

## 2. Rosie Server Hardware

The central server runs the core AI services (STT, LLM, TTS, Memory). Performance scales with hardware.

### 2.1 Minimum Configuration

**Target Use Case:** 1-2 concurrent users, basic functionality

| Component | Specification | Notes |
|-----------|---------------|-------|
| **CPU** | 4 cores @ 2.0GHz+ | x86_64 (Intel/AMD) or ARM64 |
| **RAM** | 8GB DDR4 | 4GB OS + 3GB Ollama + 1GB services |
| **Storage** | 20GB SSD | 10GB models + 5GB conversations + 5GB OS |
| **Network** | 100Mbps Ethernet or WiFi 5 | Low latency preferred |
| **GPU** | None (CPU inference) | Optional but not required |
| **OS** | Ubuntu 22.04+ / Debian 11+ | Other Linux distros supported |

**Performance Expectations:**
- Response time: 2-4 seconds
- STT latency: 1-2 seconds
- LLM inference (3B model): 1-2 seconds (CPU)
- TTS latency: 0.5-1 second
- Max concurrent clients: 2-3

**Example Hardware:**
- Intel NUC i3 (8th gen+)
- Raspberry Pi 5 (8GB) - *marginal, slower*
- Old laptop repurposed
- **Cost:** $200-400 (used/refurbished)

---

### 2.2 Recommended Configuration

**Target Use Case:** 3-5 concurrent users, responsive experience

| Component | Specification | Notes |
|-----------|---------------|-------|
| **CPU** | 6+ cores @ 3.0GHz+ | Modern Ryzen/Intel preferred |
| **RAM** | 16GB DDR4/DDR5 | 4GB OS + 8GB Ollama + 4GB services |
| **Storage** | 50GB NVMe SSD | Faster model loading, more conversation history |
| **Network** | 1Gbps Ethernet | WiFi 6 acceptable |
| **GPU** | Optional: Integrated (Radeon 660M+, Intel Iris Xe) | 2x faster inference |
| **OS** | Ubuntu 24.04 LTS | Latest LTS for best support |

**Performance Expectations:**
- Response time: 1-2 seconds
- STT latency: <500ms
- LLM inference (3B model): <1 second (GPU) or 1-2s (CPU)
- TTS latency: <300ms
- Max concurrent clients: 5-7

**Example Hardware:**
- **Beelink SER6** (Ryzen 5 6600H, 16GB) - $400
- **Intel NUC 12** (i5-1240P, 16GB) - $500
- **Mac Mini M2** (16GB) - $800 (excellent performance)
- **Custom build** (Ryzen 5 5600 + 16GB RAM) - $400

**This is the sweet spot for most users.**

---

### 2.3 Optimal Configuration

**Target Use Case:** 10+ concurrent users, multiple rooms, heavy integrations

| Component | Specification | Notes |
|-----------|---------------|-------|
| **CPU** | 8+ cores @ 3.5GHz+ | Ryzen 9 / Intel i7-i9 |
| **RAM** | 32GB DDR5 | Headroom for larger models |
| **Storage** | 100GB+ NVMe SSD | Multiple models cached |
| **Network** | 2.5Gbps Ethernet | Low latency, high throughput |
| **GPU** | Integrated (Radeon 780M) or discrete | Significant speedup |
| **OS** | Ubuntu 24.04 LTS | |

**Performance Expectations:**
- Response time: <1 second
- STT latency: <300ms
- LLM inference (3B model): <500ms (GPU)
- TTS latency: <200ms
- Max concurrent clients: 10-15

**Example Hardware:**
- **Mini PC (Ryzen 9 7940HS, 32GB)** - $600-800 ← *Your "New Rosie"*
- **Mac Studio M2 Max** - $2000 (overkill but excellent)
- **Custom server** (Ryzen 9 5900X + 32GB) - $800

---

## 3. Optional Network AI Server

For complex queries requiring large models (70B+ parameters), a separate GPU server can be used.

### 3.1 Minimum GPU Server

**Target:** Run quantized 70B model (Q4)

| Component | Specification | Notes |
|-----------|---------------|-------|
| **GPU** | NVIDIA RTX 3060 (12GB) | Minimum for 70B Q4 |
| **CPU** | 4+ cores | Mostly idle, GPU does work |
| **RAM** | 16GB | Model offloading if needed |
| **Storage** | 50GB SSD | Model storage |
| **Network** | 1Gbps Ethernet to Rosie | Low latency critical |

**Models Supported:**
- Llama 3.3 70B (Q4_K_M quantization)
- Mixtral 8x7B (Q5)
- Qwen 2.5 72B (Q4)

**Performance:**
- Inference speed: 10-20 tokens/sec
- Cold start: 5-10 seconds (model loading)
- Warm queries: <3 seconds

---

### 3.2 Recommended GPU Server

**Target:** Run multiple models, faster inference

| Component | Specification | Notes |
|-----------|---------------|-------|
| **GPU** | NVIDIA RTX 3090 (24GB) or RTX 4090 (24GB) | 2x faster, more headroom |
| **CPU** | 6+ cores | |
| **RAM** | 32GB | Comfortable |
| **Storage** | 100GB+ SSD | Multiple large models |
| **Network** | 1Gbps Ethernet | |

**Models Supported:**
- Llama 3.3 70B (Q5 or Q6) - Higher quality
- Multiple 7B-13B models simultaneously
- Qwen 2.5 72B (Q5)

**Performance:**
- Inference speed: 30-50 tokens/sec
- Response time: <2 seconds
- Can serve multiple Rosie instances

**Example Hardware:**
- Used gaming PC with RTX 3090 - $1200-1500
- **Your "Sofia" setup** (RTX 3060 + 3090) - Excellent

---

## 4. Client Endpoint Hardware

### 4.1 Jailbroken Echo Show (Gen 1)

**Specifications:**
- **Display:** 7" touchscreen (1024x600)
- **CPU:** MediaTek MT8163 (quad-core, 1.5GHz)
- **RAM:** 1GB
- **Audio:** 7-mic far-field array, decent speakers
- **Network:** WiFi 802.11ac (WiFi 5)
- **OS:** LineageOS 14.1 (Android 7)

**Pros:**
- ✅ Purpose-built for voice (excellent mic array)
- ✅ Compact, desk-friendly form factor
- ✅ Cheap ($20-40 on eBay)
- ✅ Decent display and speakers
- ✅ Built-in touchscreen

**Cons:**
- ❌ Limited RAM (1GB) - browser can be sluggish
- ❌ Old Android version (security updates limited)
- ❌ Requires jailbreaking (voids any warranty)

**Best For:** Primary voice interaction device in key rooms (office, kitchen)

**Setup Guide:** See `/endpoints/echo-show/README.md`

---

### 4.2 Raspberry Pi Display

**Minimum:**
- **Raspberry Pi 4B (4GB)** or **Pi 5 (4GB)**
- **Display:** Official 7" touchscreen or HDMI monitor
- **Audio:** USB microphone + USB/3.5mm speakers
- **Storage:** 16GB microSD (Class 10)
- **Network:** Ethernet or WiFi 5+

**Recommended:**
- **Raspberry Pi 5 (8GB)** - Faster browser, smoother UI
- **ReSpeaker 2-Mics HAT** - Better mic quality than USB
- **32GB microSD** - Room for logs, cache

**Pros:**
- ✅ Highly customizable
- ✅ Strong community support
- ✅ No jailbreaking needed
- ✅ Can add camera, sensors, etc.

**Cons:**
- ❌ Requires assembly (Pi + display + mic + speaker)
- ❌ More cables, less polished
- ❌ Higher cost than used Echo Show (~$100+ total)

**Best For:** DIY enthusiasts, custom form factors, expansion projects

---

### 4.3 Generic Tablet/PC (Web Browser)

**Minimum:**
- **Any device with modern web browser** (Chromium, Firefox, Safari)
- **RAM:** 2GB+
- **Display:** Any size
- **Microphone & Speakers:** Built-in or USB
- **Network:** WiFi or Ethernet

**Examples:**
- Old Android tablet (Fire HD 8+, Samsung Tab)
- Old iPad (iOS 14+)
- Laptop (Windows, Mac, Linux)
- Desktop PC with webcam/mic

**Pros:**
- ✅ No special hardware needed
- ✅ Works on devices you already own
- ✅ No setup beyond opening browser

**Cons:**
- ❌ Not always-on (unless configured)
- ❌ General-purpose device (distractions)

**Best For:** Testing, secondary locations, mobile use

---

## 5. Network Infrastructure

### 5.1 Network Requirements

**Minimum:**
- **Router:** 802.11ac (WiFi 5) or Ethernet
- **Bandwidth:** 100Mbps
- **Latency:** <50ms to Rosie server

**Recommended:**
- **Router:** 802.11ax (WiFi 6) or wired Gigabit Ethernet
- **Bandwidth:** 1Gbps
- **Latency:** <10ms to Rosie server
- **VLAN (optional):** Isolate IoT devices from main network

**Considerations:**
- **Echo Shows:** WiFi only (5GHz preferred for lower latency)
- **Raspberry Pi:** Ethernet strongly recommended
- **GPU Server:** Ethernet mandatory (WiFi will bottleneck)

---

### 5.2 Network Topology

**Recommended Setup:**

```
┌─────────────────────────────────────────────┐
│         Home Router (192.168.1.1)           │
│  • 1Gbps Ethernet ports                      │
│  • WiFi 6 (5GHz preferred)                   │
└──────────────┬──────────────────────────────┘
               │
    ┌──────────┼────────────┬─────────────┐
    │          │            │             │
┌───▼───┐  ┌───▼───┐   ┌───▼────┐   ┌────▼────┐
│ Rosie │  │ Sofia │   │ Echo   │   │ Echo    │
│Server │  │ GPU   │   │ Show 1 │   │ Show 2  │
│(Eth)  │  │Server │   │(WiFi 6)│   │(WiFi 6) │
└───────┘  │(Eth)  │   └────────┘   └─────────┘
           └───────┘
```

**Tips:**
- Place Rosie server and GPU server on Ethernet (not WiFi)
- Use 5GHz WiFi for Echo Shows (less interference)
- Keep Echo Shows within good WiFi range (no dead zones)
- Consider mesh WiFi if house is large

---

## 6. Storage Requirements

### 6.1 Rosie Server Storage Breakdown

**Base Installation (~10GB):**
- OS (Ubuntu Server): ~3GB
- Docker images: ~2GB
- Python dependencies: ~1GB
- System overhead: ~4GB

**AI Models (~10-30GB):**
- Faster-Whisper (STT): ~1-3GB per model (base, small, medium)
- Ollama 3B LLM: ~2GB
- Ollama 7B LLM: ~4GB (optional)
- Piper TTS voices: ~50-100MB per voice
- Embeddings model (for ChromaDB): ~500MB

**Data (Grows Over Time):**
- Conversation history: ~10MB/1000 messages
- ChromaDB embeddings: ~50MB/10k messages
- Logs: ~1GB/month (with rotation)

**Recommendation:**
- Start with 50GB
- Plan for 100GB+ if storing years of conversations
- Use SSD (not HDD) for model loading speed

---

### 6.2 GPU Server Storage

**Models (~50-200GB):**
- Llama 3.3 70B (Q4): ~40GB
- Llama 3.3 70B (Q6): ~60GB
- Mixtral 8x7B: ~30GB
- Multiple 7B-13B models: ~5-10GB each

**Recommendation:**
- Minimum: 100GB SSD
- Comfortable: 256GB SSD
- Enthusiast: 500GB+ (store many models)

---

## 7. Power & Cooling

### 7.1 Power Consumption

**Rosie Server (Idle):**
- Mini PC (Ryzen 5 6600H): ~15-25W
- Mini PC (Ryzen 9 7940HS): ~20-30W
- Raspberry Pi 5: ~5-8W
- Intel NUC: ~10-20W

**Rosie Server (Active - inference):**
- CPU inference (3B model): +20-40W
- GPU inference (integrated): +15-30W

**GPU Server (Idle):**
- RTX 3060: ~20-30W
- RTX 3090: ~30-50W

**GPU Server (Active - inference):**
- RTX 3060: ~170W (under load)
- RTX 3090: ~350W (under load)

**Client Endpoints:**
- Echo Show Gen 1: ~5-10W
- Raspberry Pi 4/5: ~5-15W

**Total System (24/7):**
- Rosie server only: 15-30W idle (~$3-6/month)
- Rosie + GPU server: 50-80W idle (~$10-15/month)
- With clients: +20W (~$4/month for 4 Shows)

---

### 7.2 Cooling Considerations

**Rosie Server:**
- Mini PCs have active cooling (fans)
- Ensure good airflow (don't stack/enclose)
- Keep ambient temp <30°C (86°F)
- Monitor CPU temps (<80°C under load)

**GPU Server:**
- RTX 3090 runs hot (80-85°C under load is normal)
- Ensure case has good airflow
- Consider fan curve adjustments
- Monitor GPU temps via `nvidia-smi`

**Client Endpoints:**
- Echo Show: Passive cooling (can get warm)
- Raspberry Pi: Add heatsinks or small fan

---

## 8. Hardware Comparison Matrix

### 8.1 Rosie Server Options

| Hardware | CPU | RAM | Storage | GPU | Price | Performance | Best For |
|----------|-----|-----|---------|-----|-------|-------------|----------|
| **RPi 5 (8GB)** | 4-core ARM | 8GB | 32GB SD | None | $80 | ⭐⭐ | Budget, testing |
| **Beelink SER6** | Ryzen 5 6600H | 16GB | 500GB | 660M | $400 | ⭐⭐⭐⭐ | Recommended |
| **Mini PC (Ryzen 9)** | Ryzen 9 7940HS | 32GB | 1TB | 780M | $700 | ⭐⭐⭐⭐⭐ | Optimal |
| **Intel NUC 12** | i5-1240P | 16GB | 500GB | Iris Xe | $500 | ⭐⭐⭐⭐ | Balanced |
| **Mac Mini M2** | M2 (8-core) | 16GB | 256GB | Unified | $800 | ⭐⭐⭐⭐⭐ | Mac users |

---

### 8.2 Client Endpoint Options

| Hardware | Display | Mic | Speaker | Network | Price | Difficulty | Best For |
|----------|---------|-----|---------|---------|-------|------------|----------|
| **Echo Show Gen 1** | 7" Touch | 7-mic array | Good | WiFi 5 | $30 | ⭐⭐⭐ | Best UX |
| **Raspberry Pi Kit** | External | USB/HAT | USB/3.5mm | WiFi/Eth | $100 | ⭐⭐⭐⭐ | DIY |
| **Fire HD 8** | 8" Touch | Built-in | OK | WiFi 5 | $50 | ⭐ | Budget tablet |
| **Old Android Tablet** | Varies | Built-in | Varies | WiFi | Free | ⭐ | Repurpose |
| **Old Laptop** | Built-in | Built-in | Built-in | WiFi/Eth | Free | ⭐ | Testing |

**Legend:**
- ⭐ = Very Easy
- ⭐⭐ = Easy
- ⭐⭐⭐ = Moderate (requires jailbreak/setup)
- ⭐⭐⭐⭐ = Advanced (assembly required)
- ⭐⭐⭐⭐⭐ = Expert

---

## 9. Upgrade Paths

### 9.1 Starting Small → Growing

**Phase 1: Minimum viable (Budget: $100-200)**
- Raspberry Pi 5 (8GB) as Rosie server
- 1 client (web browser on existing device)
- **Capabilities:** Basic voice, 1-2 users, slower responses

**Phase 2: Better experience (Budget: +$300-400)**
- Upgrade to Beelink SER6 or similar mini PC
- Add 1 jailbroken Echo Show
- **Capabilities:** 3-5 users, faster responses, better UX

**Phase 3: Heavy usage (Budget: +$500-1000)**
- Add GPU server (used PC with RTX 3060)
- Add 2-3 more Echo Shows (multiple rooms)
- **Capabilities:** 10+ users, complex queries, whole-home coverage

**Phase 4: Enthusiast (Budget: +$1000+)**
- Upgrade GPU to RTX 3090 or 4090
- Upgrade Rosie server to Ryzen 9 + 32GB
- Echo Shows in every room
- **Capabilities:** Unlimited headroom, multiple large models

---

## 10. Buying Guide

### 10.1 Where to Buy

**New:**
- **Mini PCs:** Amazon, Newegg, Beelink official store
- **Raspberry Pi:** Official resellers (check rpilocator.com for stock)
- **GPUs:** Newegg, Amazon, Micro Center

**Used/Refurbished:**
- **Echo Show Gen 1:** eBay ($20-40) - *Best value*
- **Mini PCs:** eBay, Craigslist, Facebook Marketplace
- **GPUs:** eBay, r/hardwareswap, local gaming groups
- **Raspberry Pi:** Less common used, buy new

---

### 10.2 What to Avoid

**For Rosie Server:**
- ❌ Raspberry Pi 3 or older (too slow)
- ❌ Less than 8GB RAM (swapping kills performance)
- ❌ Hard disk drive (HDD) - models load too slowly
- ❌ Single-core or dual-core CPUs

**For GPU Server:**
- ❌ GPUs with <12GB VRAM (can't run 70B models)
- ❌ Old GTX 1000-series (no FP16, slow for AI)
- ❌ Crypto-mined cards (potential reliability issues)

**For Client Endpoints:**
- ❌ Echo Show Gen 2+ (harder to jailbreak, not worth it)
- ❌ Very old tablets (<2GB RAM, Android <7)

---

## 11. Hardware Benchmarks

### 11.1 Rosie Server Performance

**3B Model Inference (100 tokens):**
| Hardware | CPU Time | GPU Time | Tokens/sec |
|----------|----------|----------|------------|
| RPi 5 (8GB) | 8-10s | N/A | 10 |
| Beelink SER6 (CPU) | 3-4s | N/A | 25 |
| Beelink SER6 (iGPU) | N/A | 1.5-2s | 50 |
| Ryzen 9 (CPU) | 2-3s | N/A | 35 |
| Ryzen 9 (iGPU) | N/A | 1-1.5s | 70 |
| Mac Mini M2 | 1.5-2s | N/A | 60 |

**STT (Faster-Whisper, 5s audio):**
| Hardware | Processing Time |
|----------|-----------------|
| RPi 5 (CPU) | 2-3s |
| Beelink SER6 (CPU) | 0.8-1s |
| Beelink SER6 (iGPU) | 0.3-0.5s |
| Ryzen 9 (iGPU) | 0.2-0.3s |

---

### 11.2 GPU Server Performance

**70B Model Inference (100 tokens, Q4 quantization):**
| GPU | Tokens/sec | Response Time |
|-----|------------|---------------|
| RTX 3060 (12GB) | 10-15 | 6-10s |
| RTX 3090 (24GB) | 25-35 | 3-4s |
| RTX 4090 (24GB) | 45-60 | 1.5-2s |

---

## 12. Future-Proofing

**Considerations:**
- **RAM:** 16GB is comfortable now, 32GB ensures headroom for future models
- **Storage:** NVMe SSD speeds up model loading (vs SATA SSD)
- **Network:** WiFi 6E/7 emerging, but WiFi 6 sufficient for years
- **GPU VRAM:** 12GB minimum, 24GB preferred for future 100B+ models

**Expected Lifespan:**
- **Rosie Server:** 3-5 years (CPU/RAM limits eventually)
- **GPU Server:** 5-7 years (new models require more VRAM over time)
- **Client Endpoints:** Echo Shows/Pis last 5+ years (just display/audio)

---

## 13. FAQ

**Q: Can I run Rosie on a Raspberry Pi?**  
A: Yes, but Pi 5 (8GB) minimum. Responses will be slower (~3-5s). Good for testing, marginal for daily use.

**Q: Do I need a GPU?**  
A: No. CPU inference works. GPU (integrated or discrete) speeds things up 2-5x.

**Q: Can I use an old gaming PC as the server?**  
A: Absolutely! If it has 6+ cores and 16GB RAM, it's perfect.

**Q: Can I run Rosie and the GPU server on the same machine?**  
A: Yes, if you have a discrete GPU (RTX 3060+). Rosie runs on CPU/iGPU, large models on dGPU.

**Q: How much does it cost to run 24/7?**  
A: ~$5-15/month in electricity (depending on hardware and local rates).

**Q: Can I use an old Echo Show from a thrift store?**  
A: Yes! As long as it's Gen 1 and powers on, jailbreak it and you're golden.

---

## End of Hardware Guide

**Recommended Starting Point:**
- **Server:** Beelink SER6 or similar mini PC ($400)
- **Client:** 1 jailbroken Echo Show ($30 on eBay)
- **Total:** ~$450 for complete system

**Next Steps:**
1. Choose your hardware tier (budget, recommended, or optimal)
2. Order components
3. Follow setup guide in [DEVELOPMENT.md](DEVELOPMENT.md)
