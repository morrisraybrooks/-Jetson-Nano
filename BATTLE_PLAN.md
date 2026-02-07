# 🎯 Jetson Orin Nano Super - AI Coding Machine Battle Plan

## The Vision

Build a dedicated AI coding assistant powered by the NVIDIA Jetson Orin Nano Super Developer Kit, interfaced with an ODROID-H3 desktop workstation. The Jetson handles all AI inference while the ODROID remains the primary development machine. Together they create a local, private, uncensored, subscription-free alternative to GitHub Copilot and Augment Code.

---

## Hardware Inventory

### Machine 1: ODROID-H3 (Desktop Workstation)
| Component | Details |
|-----------|---------|
| CPU | Intel Pentium Silver N6005 (4-core, 3.3GHz boost) |
| RAM | 32GB DDR4 |
| Storage | 1TB WD Blue SN570 NVMe (KEEP IN THIS MACHINE) |
| Network | Dual 2.5GbE (RTL8125) - one port unused |
| OS | Ubuntu 25.10 |
| Role | Primary coding machine, VS Code, file storage |

### Machine 2: Jetson Orin Nano Super (AI Brain)
| Component | Details |
|-----------|---------|
| GPU | NVIDIA Ampere - 1024 CUDA cores, 32 Tensor cores |
| AI Performance | 67 TOPS (INT8) |
| CPU | 6-core ARM Cortex-A78AE |
| RAM | 8GB LPDDR5 (shared CPU/GPU) @ 102 GB/s |
| Storage | NEED TO BUY: 256GB+ NVMe SSD |
| Power | 7W-25W configurable |
| Role | Dedicated AI inference, Agent Zero, model serving |

### Shopping List
- [ ] NVMe SSD for Jetson (256GB minimum, 512GB ideal) - ~$25-40
- [ ] Cat6 Ethernet cable (for direct ODROID-to-Jetson link) - ~$5-10
- [ ] Active cooling fan/heatsink for Jetson (if not included) - ~$10-15

---

## Network Architecture

```
Internet
   │
   ▼
[Router 192.168.12.x]
   │
   ├── ODROID-H3 enp1s0 (192.168.12.147) ── Main network + internet
   │
   └── Jetson Orin Nano (192.168.12.x) ──── Main network + internet
          │
          │ (Direct 2.5GbE cable - dedicated fast link)
          │
   ODROID-H3 enp2s0 ◄──────────────────────► Jetson eth1
   (10.0.0.1)                                 (10.0.0.2)

Tailscale Mesh (backup + remote access):
   ODROID:        100.66.226.32
   Jetson:        (will be assigned on setup)
   Home Assistant: 100.105.165.29
```

### Connection Methods (Priority Order)
1. **Direct Ethernet** (10.0.0.x) - Fastest, for file sharing + API calls
2. **LAN** (192.168.12.x) - Standard network access
3. **Tailscale** - Backup, remote access, works from anywhere

---

## Software Stack

### Jetson Orin Nano Super
```
┌─────────────────────────────────────────┐
│  JetPack 6.x (Ubuntu 22.04 based)      │
├─────────────────────────────────────────┤
│  Ollama (model serving)                 │
│  ├── Qwen 2.5 Coder 7B (Q4_K_M)       │  ◄── Primary coding model
│  ├── Dolphin Mistral 7B                 │  ◄── Uncensored general model
│  └── Stable Diffusion 1.5              │  ◄── Image gen (swap in when needed)
├─────────────────────────────────────────┤
│  Agent Zero Framework                   │
│  ├── Persistent memory (ChromaDB)       │
│  ├── File access (NFS mount to ODROID)  │
│  ├── SSH access to ODROID               │
│  ├── Internet access                    │
│  └── Command execution                  │
├─────────────────────────────────────────┤
│  Tailscale (mesh networking)            │
│  SSH Server (for management)            │
│  NFS Client (mount ODROID files)        │
│  16GB Swap on NVMe                      │
└─────────────────────────────────────────┘
```

### ODROID-H3 (Desktop)
```
┌─────────────────────────────────────────┐
│  Ubuntu 25.10                           │
├─────────────────────────────────────────┤
│  VS Code                                │
│  └── Continue.dev extension             │
│      └── Points to Jetson Ollama API    │
│          (http://10.0.0.2:11434)        │
├─────────────────────────────────────────┤
│  NFS Server (shares /home/morris)       │
│  SSH Server (Agent Zero connects back)  │
│  Tailscale (already installed)          │
│  Docker (already installed)             │
│  Samba (already installed, activate)    │
└─────────────────────────────────────────┘
```

---

## AI Models Strategy

### Primary: Qwen 2.5 Coder 7B Instruct (Q4_K_M quantization)
- **Purpose**: Day-to-day coding assistant (Copilot replacement)
- **Size**: ~4.5GB in RAM
- **Strengths**: Python, C++, Java, Shell, HTML - all your languages
- **Used by**: Continue.dev (autocomplete + chat) and Agent Zero

### Secondary: Dolphin Mistral 7B (uncensored)
- **Purpose**: Unrestricted general assistant, security research, creative tasks
- **Size**: ~4.5GB in RAM
- **Strengths**: Zero refusals, strong reasoning, good at everything
- **Used by**: Agent Zero for tasks that censored models refuse

### Optional: Stable Diffusion 1.5 (uncensored fine-tunes)
- **Purpose**: Image generation when you want it
- **Size**: ~2-3GB in RAM
- **Note**: Must swap out coding model first, ~30-60 sec per image
- **Used by**: Swap in on demand with a simple command

### Model Swapping
```bash
# Quick aliases to set up:
alias codingmode='ollama stop dolphin-mistral 2>/dev/null; ollama run qwen2.5-coder:7b-instruct-q4_K_M'
alias hackmode='ollama stop qwen2.5-coder 2>/dev/null; ollama run dolphin-mistral'
alias artmode='ollama stop qwen2.5-coder 2>/dev/null; ollama stop dolphin-mistral 2>/dev/null; ollama run stable-diffusion'
```

---

## Agent Zero Capabilities

Once set up, Agent Zero on the Jetson will be able to:

| Capability | How |
|-----------|-----|
| **Read your files** | NFS mount of ODROID /home/morris |
| **Edit your files** | Direct file write over NFS |
| **Run commands on ODROID** | SSH from Jetson → ODROID |
| **Run commands on Jetson** | Local terminal execution |
| **Browse the internet** | Direct internet access |
| **Search the web** | Built-in web search tools |
| **Remember everything** | ChromaDB persistent vector memory |
| **Know your projects** | RAG indexing of your codebase |
| **Generate images** | Swap to SD model on demand |
| **Penetration testing** | Uncensored model + full tool access |
| **Build full projects** | Autonomous multi-file code generation |

---

## Build Phases

### Phase 1: Hardware Setup (Day 1)
- [ ] Buy NVMe SSD for Jetson (256-512GB)
- [ ] Buy Cat6 ethernet cable
- [ ] Verify Jetson cooling solution
- [ ] Install NVMe into Jetson
- [ ] Flash JetPack 6.x onto Jetson
- [ ] Boot and verify Jetson works
- [ ] Set up partitions:
  - 50GB - OS
  - 16GB - Swap
  - Remaining - Data/Models

### Phase 2: Network Setup (Day 1-2)
- [ ] Connect Jetson to router (get internet)
- [ ] Connect direct ethernet cable: ODROID enp2s0 ↔ Jetson
- [ ] Configure static IPs on direct link (10.0.0.1 / 10.0.0.2)
- [ ] Install Tailscale on Jetson
- [ ] Verify all three connection paths work
- [ ] Enable SSH server on both machines
- [ ] Test SSH both directions

### Phase 3: File Sharing (Day 2)
- [ ] Set up NFS server on ODROID (share /home/morris)
- [ ] Mount ODROID files on Jetson via NFS
- [ ] Test read/write from Jetson to ODROID files
- [ ] Set up auto-mount on boot (fstab)
- [ ] Verify permissions work correctly

### Phase 4: AI Stack on Jetson (Day 2-3)
- [ ] Install Ollama on Jetson (ARM64/CUDA)
- [ ] Download Qwen 2.5 Coder 7B (Q4_K_M)
- [ ] Download Dolphin Mistral 7B
- [ ] Test model inference and speed
- [ ] Benchmark tokens/sec
- [ ] Set Ollama to start on boot
- [ ] Configure Ollama to listen on all interfaces (0.0.0.0)
- [ ] Test API access from ODROID (curl http://10.0.0.2:11434)

### Phase 5: Agent Zero on Jetson (Day 3-4)
- [ ] Clone Agent Zero repo to Jetson
- [ ] Configure to use local Ollama (no API keys needed)
- [ ] Set up persistent memory (ChromaDB)
- [ ] Configure file access to ODROID mount
- [ ] Configure SSH tool (Jetson → ODROID command execution)
- [ ] Configure internet/web browsing tools
- [ ] Set up uncensored system prompts
- [ ] Test autonomous task completion
- [ ] Set Agent Zero to start on boot

### Phase 6: VS Code Integration on ODROID (Day 4)
- [ ] Install Continue.dev extension in VS Code
- [ ] Configure Continue.dev to point to Jetson Ollama API
- [ ] Test inline autocomplete
- [ ] Test chat sidebar
- [ ] Test code actions (explain, refactor, debug)
- [ ] Tune settings for best experience

### Phase 7: Polish and Optimize (Day 5+)
- [ ] Set up model swap aliases (codingmode/hackmode/artmode)
- [ ] Index your existing projects into RAG (persistent memory)
- [ ] Fine-tune Agent Zero prompts for your workflow
- [ ] Set up monitoring (jtop on Jetson, resource usage)
- [ ] Create startup scripts for everything
- [ ] Test image generation (Stable Diffusion swap)
- [ ] Stress test: run coding AI while working in VS Code
- [ ] Document any issues and solutions

---

## Daily Workflow (Once Built)

```
Morning:
1. Both machines are already running (always on)
2. Sit down at ODROID, open VS Code
3. Continue.dev is already connected to Jetson
4. Start coding with AI autocomplete

Need help:
1. Ask Continue.dev chat sidebar (quick questions)
2. Open Agent Zero web UI for complex tasks
3. "Build me a web scraper for X" → Agent Zero does it autonomously
4. Agent Zero reads your files, writes code, runs tests

Need uncensored:
1. Switch to Dolphin model: `hackmode`
2. Ask whatever you want
3. Switch back: `codingmode`

Need images:
1. Switch to SD: `artmode`
2. Generate whatever you want
3. Switch back: `codingmode`

Security research:
1. Agent Zero + Dolphin model
2. Full penetration testing toolkit
3. No restrictions, no refusals
```

---

## Security & Counter-Security Lab

This setup doubles as a personal cybersecurity research lab. The Jetson runs uncensored models that won't refuse security-related requests, and Agent Zero already has a red team operative system prompt. The goal is to experiment freely with both offensive and defensive security.

### 🔴 Offensive Security (Red Team)

**AI-Assisted Penetration Testing:**
- Agent Zero + Dolphin Mistral = unrestricted pen test assistant
- Generate exploit code, shellcode, payloads without refusals
- Automate recon: "Scan this network and find every open service"
- Write custom Nmap scripts, Metasploit modules, Burp extensions
- Social engineering template generation (phishing emails, pretexting scripts)

**Tools to Install on Jetson/ODROID:**
- [ ] Nmap (network scanning & enumeration)
- [ ] Metasploit Framework (exploitation)
- [ ] Burp Suite Community (web app testing)
- [ ] SQLmap (SQL injection automation)
- [ ] Hydra (brute force / credential testing)
- [ ] Impacket (Windows protocol attacks)
- [ ] Responder (LLMNR/NBT-NS poisoning)
- [ ] Aircrack-ng (wireless security testing)
- [ ] John the Ripper / Hashcat (password cracking)
- [ ] Nikto (web server scanner)
- [ ] Gobuster / ffuf (directory & DNS brute forcing)
- [ ] Wireshark / tcpdump (packet analysis)
- [ ] Scapy (custom packet crafting via Python)
- [ ] Pwntools (CTF & exploit development framework)

**AI-Powered Attack Workflows:**
```
1. "Agent Zero, scan 192.168.12.0/24 and enumerate all services"
   → Runs Nmap, parses output, identifies targets

2. "Find vulnerabilities in the web app at 192.168.12.50:8080"
   → Runs Nikto + SQLmap + Gobuster, reports findings

3. "Write a Python reverse shell that evades Windows Defender"
   → Dolphin Mistral generates it without refusal

4. "Crack these hashes from the database dump"
   → Runs John/Hashcat, reports results

5. "Generate a spear-phishing email targeting an IT admin"
   → Creates realistic template with payload suggestions
```

### 🔵 Defensive Security (Blue Team)

**AI-Assisted Defense & Hardening:**
- Analyze logs for indicators of compromise (IOCs)
- Write firewall rules (iptables/nftables/ufw)
- Generate IDS/IPS signatures (Snort/Suricata rules)
- Audit system configurations for weaknesses
- Automate security patching scripts
- Review code for vulnerabilities (SAST-style analysis)

**Tools to Install:**
- [ ] Suricata / Snort (intrusion detection)
- [ ] OSSEC / Wazuh (host-based IDS)
- [ ] Fail2ban (brute force protection)
- [ ] ClamAV (malware scanning)
- [ ] Lynis (system security auditing)
- [ ] OpenVAS / Greenbone (vulnerability scanning)
- [ ] YARA (malware pattern matching)

**AI-Powered Defense Workflows:**
```
1. "Analyze /var/log/auth.log for brute force attempts"
   → Parses logs, identifies IPs, generates Fail2ban rules

2. "Harden this Ubuntu server - check everything"
   → Runs Lynis, reviews output, applies fixes

3. "Write Suricata rules to detect this attack pattern"
   → Generates custom IDS signatures

4. "Review this Python web app for security vulnerabilities"
   → Reads code over NFS, identifies SQLi/XSS/SSRF issues

5. "Set up a honeypot to catch network intruders"
   → Deploys and configures a basic honeypot service
```

### 🟣 Purple Team (Attack + Defend Cycle)

The real power is running both sides:

```
┌─────────────────────────────────────────────────┐
│              PURPLE TEAM CYCLE                  │
│                                                 │
│   1. ATTACK: Find a vulnerability               │
│      ↓                                          │
│   2. DOCUMENT: Log the exploit chain            │
│      ↓                                          │
│   3. DEFEND: Write detection rules              │
│      ↓                                          │
│   4. VERIFY: Re-attack to test the defense      │
│      ↓                                          │
│   5. HARDEN: Patch the vulnerability            │
│      ↓                                          │
│   6. REPEAT: Find the next weakness             │
└─────────────────────────────────────────────────┘
```

**Example Purple Team Exercise:**
- Attack your own Home Assistant (100.105.165.29) from the Jetson
- Find misconfigurations, weak APIs, exposed services
- Write Suricata rules to detect the attack
- Harden Home Assistant based on findings
- Re-test to verify the fix works

### 🧪 CTF & Training

- [ ] Set up VulnHub/HackTheBox VMs on ODROID (32GB RAM = plenty)
- [ ] Use Agent Zero to assist with CTF challenges
- [ ] Practice OSCP-style boxes with AI guidance
- [ ] Study CVEs: "Explain CVE-2024-XXXX and write a PoC"
- [ ] Reverse engineering with Ghidra (ODROID) + AI analysis (Jetson)

### 🔑 Security Lab Network Isolation

**IMPORTANT: Keep lab traffic isolated from production:**
- [ ] Use the direct ethernet link (10.0.0.x) for attack traffic
- [ ] Set up VLANs or separate subnets for vulnerable VMs
- [ ] Never run offensive tools against targets you don't own
- [ ] Use Docker containers for disposable attack/target environments
- [ ] Snapshot VMs before testing so you can roll back

### Security Mode Aliases
```bash
# Add to ~/.bashrc on Jetson:
alias recon='hackmode && echo "🔴 Red Team mode active - Dolphin loaded"'
alias defend='codingmode && echo "🔵 Blue Team mode active - Qwen loaded"'
alias purpleteam='hackmode && echo "🟣 Purple Team mode - full offense + defense"'
```

---

## Performance Expectations

| Task | Expected Speed |
|------|---------------|
| Autocomplete (single line) | 1-3 seconds |
| Chat response (paragraph) | 5-15 seconds |
| Agent Zero task (build a script) | 1-5 minutes |
| Image generation (512x512) | 30-60 seconds |
| Model swap | 10-15 seconds |
| File access over NFS | Near instant |
| SSH command execution | < 1 second |

---

## Cost Summary

| Item | Cost |
|------|------|
| Jetson Orin Nano Super (already have) | $0 |
| ODROID-H3 (already have) | $0 |
| NVMe SSD 256-512GB (need to buy) | $25-40 |
| Cat6 Ethernet cable (need to buy) | $5-10 |
| Cooling (if needed) | $10-15 |
| **Total** | **$40-65** |
| Monthly subscription cost | **$0 forever** |

---

## Notes

- The ODROID stays as the primary workstation - don't rob its 1TB NVMe
- The Jetson is a dedicated AI brain - headless or minimal UI
- Tailscale gives you access from anywhere (phone, laptop, etc.)
- All models are uncensored - your hardware, your rules
- Agent Zero has full system access - files, commands, internet, memory
- This setup can grow: add more Jetsons, bigger models, fine-tuning later
