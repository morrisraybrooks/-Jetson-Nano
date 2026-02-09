# 🎯 Jetson Orin Nano Super - AI Coding Machine

## Architecture

**ODROID-H3** (32GB RAM, 1TB NVMe) - Development workstation running VS Code, Continue.dev, and Agent Zero
**Jetson Orin Nano** (8GB RAM) - Dedicated AI model server running Ollama

Direct connection: 10.0.0.1 (ODROID) ↔ 10.0.0.2 (Jetson) via 2.5GbE

---

## Quick Reference

### URLs
- **Agent Zero**: http://localhost:8080
- **Ollama API (Local)**: http://localhost:11434 (temporary - CPU only)
- **Ollama API (Jetson)**: http://10.0.0.2:11434 (when Jetson arrives)

### SSH
```bash
ssh jetson@10.0.0.2
```

### Available Ollama Models (Currently on ODROID)

**Best for Coding:**
- `codellama:7b-instruct` - Code generation and debugging
- `codellama:7b-python` - Python-specific coding

**Best for General Use:**
- `llama3.1:8b` - Latest Meta model, excellent reasoning (Q4_K_M)
- `llama3:latest` - Stable general-purpose model
- `mistral:latest` - Strong performance, good for complex tasks

**Uncensored Models:**
- `dolphin-llama3:latest` - Uncensored Llama 3 variant
- `llama2-uncensored:7b` - No content filtering
- `unrestricted-clean:latest` - Custom uncensored model
- `zero-limits:latest` - Maximum freedom model

**Specialized:**
- `nomic-embed-text:latest` - Text embeddings for RAG/memory
- `gemma2:latest` - Google's lightweight model
- `tinyllama:latest` - Ultra-fast, minimal resources (1.1B)

**Security/Pentesting:**
- `pentest-expert:latest` - Penetration testing assistant
- `cybersec-expert:latest` - Cybersecurity analysis
- `webapp-pentest:latest` - Web application security

### Model Management
```bash
# List all models
ollama list

# Check running models
ollama ps

# Stop a model
ollama stop <model-name>

# Test a model
ollama run llama3.1:8b "Hello, how are you?"
```

### Agent Zero (on ODROID)
```bash
sudo systemctl start agent-zero
sudo systemctl status agent-zero
journalctl -u agent-zero -f
```

---

## Setup Checklist

### Current Status (Pre-Jetson)
- [x] Agent Zero cloned to ~/Agent-Zero
- [x] Virtual environment created (venv312)
- [x] Dependencies installed
- [x] Ollama running locally with 26 models
- [x] Agent Zero configured to use local Ollama (CPU)
- [x] API keys configured (.env file)
- [x] Systemd service file created (/tmp/agent-zero.service)
- [x] Workspace directory created (~/workspace)

**Current Configuration:**
- Chat Model: `llama3.1:8b` @ http://localhost:11434
- Embedding Model: `nomic-embed-text:latest` @ http://localhost:11434
- Running on CPU (slow but functional)

### Hardware (When Ready)
- [ ] Buy NVMe SSD for Jetson (256-512GB) - ~$25-40
- [ ] Buy Cat6 ethernet cable - ~$5-10
- [ ] Install NVMe into Jetson
- [ ] Flash JetPack 6.2 onto Jetson

### Network
- [ ] Connect direct ethernet: ODROID enp2s0 ↔ Jetson
- [ ] Configure static IPs (10.0.0.1 / 10.0.0.2)
- [ ] Install Tailscale on Jetson
- [ ] Test connectivity: `ping 10.0.0.2`

### Jetson Setup
- [ ] Install Ollama (ARM64/CUDA)
- [ ] Download Qwen 2.5 Coder 7B (Q4_K_M)
- [ ] Download Dolphin Mistral 7B
- [ ] Configure Ollama to listen on 0.0.0.0:11434
- [ ] Set Ollama to start on boot (systemd)
- [ ] Test API: `curl http://10.0.0.2:11434/api/tags`

### ODROID Setup
- [ ] Clone Agent Zero: `git clone https://github.com/morrisraybrooks/Agent-Zero.git ~/agent-zero`
- [ ] Install dependencies: `cd ~/agent-zero && pip install -r requirements.txt`
- [ ] Configure Agent Zero (see config below)
- [ ] Set up systemd service (see config below)
- [ ] Install Continue.dev in VS Code
- [ ] Configure Continue.dev to use http://10.0.0.2:11434

---

## Agent Zero Configuration

### `~/agent-zero/.env`
```bash
OLLAMA_API_BASE=http://10.0.0.2:11434
OLLAMA_MODEL_CODING=qwen2.5-coder:7b-instruct-q4_K_M
OLLAMA_MODEL_UNCENSORED=dolphin-mistral
CHROMADB_PATH=/home/morris/agent-zero/chromadb
WORKSPACE_PATH=/home/morris/workspace
WEB_UI_PORT=8080
WEB_UI_HOST=0.0.0.0
ENABLE_FILE_TOOLS=true
ENABLE_COMMAND_TOOLS=true
ENABLE_WEB_TOOLS=true
ENABLE_CODE_TOOLS=true
```

### `/etc/systemd/system/agent-zero.service`
```ini
[Unit]
Description=Agent Zero AI Framework
After=network.target

[Service]
Type=simple
User=morris
WorkingDirectory=/home/morris/agent-zero
ExecStart=/usr/bin/python3 /home/morris/agent-zero/run.py
Restart=always
RestartSec=10
Environment="PATH=/usr/local/bin:/usr/bin:/bin"

[Install]
WantedBy=multi-user.target
```

Enable: `sudo systemctl daemon-reload && sudo systemctl enable agent-zero`

---

## Bash Aliases

### Jetson (~/.bashrc)
```bash
alias codingmode='ollama stop dolphin-mistral 2>/dev/null; ollama run qwen2.5-coder:7b-instruct-q4_K_M'
alias hackmode='ollama stop qwen2.5-coder 2>/dev/null; ollama run dolphin-mistral'
alias artmode='ollama stop qwen2.5-coder 2>/dev/null; ollama stop dolphin-mistral 2>/dev/null; ollama run stable-diffusion'
```

### ODROID (~/.bashrc)
```bash
alias jetson='ssh jetson@10.0.0.2'
alias agentweb='xdg-open http://localhost:8080'
```

---

## Troubleshooting

### Current Setup (CPU-based Ollama)

**Slow responses are normal on CPU!**
- First request: 30-60 seconds (model loading)
- Subsequent requests: 10-30 seconds
- This will be **much faster** on Jetson GPU

**Check Ollama status:**
```bash
systemctl status ollama
ollama ps
curl http://localhost:11434/api/tags
```

**Agent Zero errors:**
```bash
# Check logs
cd ~/Agent-Zero
tail -f logs/*.log

# Restart Agent Zero
# Press Ctrl+C in the terminal running run_ui.py
cd ~/Agent-Zero && source venv312/bin/activate && python run_ui.py
```

**Switch to a faster model (if too slow):**
```bash
# Edit ~/Agent-Zero/tmp/settings.json
# Change "chat_model_name" to "tinyllama:latest" (much faster, less capable)
```

### Future: Can't connect to Jetson
```bash
ping 10.0.0.2
curl http://10.0.0.2:11434/api/tags
ssh jetson@10.0.0.2 'systemctl status ollama'
```

### Future: Performance issues
```bash
# ODROID
htop
du -sh ~/Agent-Zero/memory

# Jetson
ssh jetson@10.0.0.2 'jtop'
```

### Test Ollama API
```bash
# Local (current)
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "llama3.1:8b", "prompt": "Hello", "stream": false}'

# Jetson (future)
curl -X POST http://10.0.0.2:11434/api/generate \
  -d '{"model": "qwen2.5-coder:7b-instruct-q4_K_M", "prompt": "Hello", "stream": false}'
```
