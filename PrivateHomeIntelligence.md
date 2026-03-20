# Locally Hosted AI Intelligence Service for Homes

This document outlines the technical configuration for a private, persistent AI service hosted on a central Linux desktop (RTX 3090) and accessed by household laptops.

---

## 1. Local Network Configuration

To allow household laptops to connect to the central service, identify and stabilize the host machine's address.

1. **Find the Host IP:** On the Linux Desktop, open the terminal and run `hostname -I`. Use the first IP listed (e.g., `192.168.1.50`).
2. **Assign a Static IP:** It is highly recommended to assign a "DHCP Reservation" in your router settings for the Linux Desktop to prevent connection drops after reboots.

---

## 2. The Serving Engine: Ollama

Ollama handles hardware acceleration and serves models to multiple devices simultaneously. It runs as a background system service on the Linux desktop.

### Installation and Updates
* **To Install:** Run `curl -fsSL https://ollama.com/install.sh | sh`
* **To Update:** Re-run the installation script above. It will detect the existing version and update the binary without affecting your downloaded models.

Install models:

| Capability | Model | Ollama Command | VRAM Usage |
| --- | --- | --- | --- |
| General/Vision | Qwen 3 (32B) | ollama pull qwen3:32b | ~20 GB |
| Coding/Logic | Qwen 3 Coder (32B) | ollama pull qwen3-coder:32b | ~20 GB |
| Chat/Reasoning | Nemotron 3 Nano (30B) | ollama pull nemotron-3-nano:30b | ~22 GB |
| Vision (Heavy) | Qwen 3 VL (32B) | ollama pull qwen3-vl:32b | ~23.5 GB | Danger Zone. Extremely tight. Limit context to 4k or less to avoid OOM. |
| Vision (Omni) | Qwen 3.5 (27B) | ollama pull qwen3.5:27b | ~19.0 GB | "Sweet Spot. Newer ""Omni"" architecture; native vision + faster text response." |
| Vision (Fast) | Qwen 3.5 (9B) | ollama pull qwen3.5:9b | ~7.5 GB | Speed King. Instant OCR and scans; can run alongside other models. |


### Multi-User and Persistence Setup

1. **Service Configuration:** Run `sudo systemctl edit ollama.service` and add these lines:

   ```ini
   [Service]
   Environment="OLLAMA_HOST=0.0.0.0"
   Environment="OLLAMA_NUM_PARALLEL=4"
   Environment="OLLAMA_MAX_LOADED_MODELS=1"
   Environment="OLLAMA_ORIGINS=*"
   ```
**FIXME:** Add information on what the Ollama environment variables do. Explore additional features like Flash Attention

2. **Apply Changes:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable ollama
   sudo systemctl restart ollama
   ```


---

## 3. The User Interface: Jan.AI

Jan.AI is the client application for family laptops. It provides the interface for chat, file analysis, and web research.

### Initial Setup & Dependencies

1. **Install Jan.AI:** Download from [jan.ai](https://jan.ai).
2. **Link to Server:** In **Settings > Remote Engines**, create an OpenAI-Compatible engine with the URL: `http://<LINUX_DESKTOP_IP>:11434/v1`.
3. **The Dependency Requirement (Node.js & Python):** For Web Search and File Tools to function, each laptop **must** have the following runtimes installed:
   - **Node.js (v20+):** Required for executing MCP scripts (like Puppeteer).
   - **Python (3.11+):** Required for various filesystem and data analysis tools.

### Advanced Feature Configuration

1. **Enable Tool Calling:** In **Model Settings (Gear Icon) > Model Capabilities**, toggle **Tool Calling** to ON.
2. **Model Context Protocol (MCP):**
   - Go to **Settings > Advanced** and toggle **Experimental Features** ON.
   - Go to **Settings > MCP Servers** and toggle **Allow All MCP Tool Permissions** ON.
3. **Web Search Modes:**
   - **Indexed Search (Fast):** Add the **Brave Search** or **Tavily** MCP server.
   - **Manual Research (Deep):** Add the **Puppeteer** MCP server (`npx -y @modelcontextprotocol/server-puppeteer`).

---

## 4. Useful Local Features & Proactive Mode

Configure these settings in Jan.AI to improve the experience for non-tech users.

- **Proactive Mode (Context Capture):** Go to **Settings > Assistant** and toggle **Proactive Mode** to ON. This allows the AI to "see" active windows or files to provide context without manual uploads.
- **Auto-Context Management:** In **Settings > Advanced**, enable **Auto Context Summary**. This prevents the AI from becoming "forgetful" in long chats by automatically summarizing earlier parts of the conversation.
- **Projects (Workspaces):** Create dedicated folders (e.g., "School Work"). By "Mounting" a local folder to a Project, the AI maintains a persistent index of every file in that directory.
- **Hardware-Aware Hub:** In **Settings > Hardware**, enable **VRAM Protection**. This grays out models in the hub that are too large for the RTX 3090, preventing accidental system crashes.
- **Global Hotkey:** Enable in **Settings > General** (e.g., `Alt + Space`) to allow family members to summon the AI instantly from any application.

---

## 5. Intelligence Selection (RTX 3090 Optimized)

| Model | Ollama Tag | Capabilities | Speed |
| :--- | :--- | :--- | :--- |
| **Qwen 3.5 32B (VLLM)** | `qwen3.5:32b` | **Default.** Best instruction following + **Vision/Images**. | ~50 tok/s |
| **Qwen3-Coder 32B** | `qwen3-coder:32b` | **Logic.** Best for math, coding, and data organization. | ~50 tok/s |
| **Nemotron 30B** | `nemotron:30b` | **Chat.** Fine-tuned for natural, helpful conversation. | ~55 tok/s |

### Visual Support (VLLM)

Using **Qwen 3.5**, users can drag photos or screenshots directly into Jan.AI. The model uses the GPU on the Linux Desktop to perform visual reasoning locally.
