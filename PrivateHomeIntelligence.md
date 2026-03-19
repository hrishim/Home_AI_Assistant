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
```
	ollama pull qwen3:32b
ollama pull qwen3.5:32b
```

| Capability | Model | Ollama Command | VRAM Usage |
| --- | --- | --- | --- |
| General/Vision | Qwen 3 (32B) | ollama pull qwen3:32b | ~20 GB |
| Coding/Logic | Qwen 3 Coder (32B) | ollama pull qwen3-coder:32b | ~20 GB |
| Chat/Reasoning | Nemotron 3 Nano (30B) | ollama pull nemotron-3-nano:30b | ~22 GB |
| Vision (Heavy) | Qwen 3 VL (32B) | ollama pull qwen3-vl:32b | ~23.5 GB | Danger Zone. Extremely tight. Limit context to 4k or less to avoid OOM. |
| Vision (Omni) | Qwen 3.5 (27B) | ollama pull qwen3.5:27b | ~19.0 GB | "Sweet Spot. Newer ""Omni"" architecture; native vision + faster text response." |
| Vision (Fast) | Qwen 3.5 (9B) | ollama pull qwen3.5:9b | ~7.5 GB | Speed King. Instant OCR and scans; can run alongside other models. |

Capability,Model,Ollama Command,VRAM Usage,Notes
General/Text,Qwen 3 (32B),ollama pull qwen3:32b,~20.0 GB,Standard high-quality text-only model.
Coding/Logic,Qwen 3 Coder (30B),ollama pull qwen3-coder:30b,~18.5 GB,Optimized for software engineering and math.
Reasoning,Nemotron 3 Nano (30B),ollama pull nemotron-3-nano:30b,~22.0 GB,NVIDIA's official fine-tune for reasoning tasks.

### Multi-User and Persistence Setup

1. **Service Configuration:** Run `sudo systemctl edit ollama.service` and add these lines:

   ```ini
   [Service]
   Environment="OLLAMA_HOST=0.0.0.0"
   Environment="OLLAMA_NUM_PARALLEL=4"
   Environment="OLLAMA_MAX_LOADED_MODELS=1"
   ```

2. **Apply Changes:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable ollama
   sudo systemctl restart ollama
   ```


---

## 3. The User Interface: Jan.AI

Jan.AI is the client application for family laptops. It provides the interface for text chat, PDF analysis, and web research via the remote Ollama server.

> **Known Limitation — Vision/Images:** Jan's image upload feature is explicitly designed to work only with local models or OpenAI API models. When using a remote Ollama engine, the image button in the chat input will always be grayed out. This is not a bug in your setup — it is a documented Jan limitation. For image and vision tasks, use **Open WebUI** instead (see Section 5).

### Initial Setup

1. **Install Jan.AI:** Download from [jan.ai](https://jan.ai).
2. **Link to Remote Server:** In **Settings > Model Providers**, click the **+** button and add an OpenAI-Compatible provider with the URL: `http://resnet.local:11434/v1`.
3. **Enable PDF File Analysis:** Go to **Settings**, enable **Experimental Mode**, then enable **Retrieval**. You can now upload PDF documents into any chat thread for analysis.

### MCP & Web Search Configuration

Go to **Settings > MCP Servers**. First, enable **Allow All MCP Tool Permissions** — this auto-approves tool calls without interrupting the conversation.

#### Intel Mac: Required Fix for All MCP Servers

Jan on Intel Macs has a bug where it tries to use its own bundled ARM node binary instead of the system one, causing all STDIO MCP servers to silently fail. Two rules apply to every MCP server you add:

**Rule 1 — Always use full paths for commands:**
- `npx`-based servers → use `/usr/local/bin/npx` instead of `npx`
- `uvx`-based servers → use `/usr/local/bin/uvx` instead of `uvx`

**Rule 2 — Always fully quit and restart Jan after changes:**
`Cmd+Q` to quit (not just close the window), then reopen. Toggling alone is not enough.

**One-time setup — fix npm cache permissions (run in Terminal):**
```bash
sudo chown -R $(whoami) ~/.npm
```

**Install uvx (required for Python-based MCP servers like fetch):**
```bash
brew install uv
```
Verify with `which uvx` — should return `/usr/local/bin/uvx`.

#### Filesystem MCP

Lets Jan read files from a folder on the laptop.
- Command: `/usr/local/bin/npx`
- Args: `-y`, `@modelcontextprotocol/server-filesystem`, `/Users/hrishi/JanDocs`

The third argument is the folder Jan is allowed to access. Change it to any folder you want. The toggle will fail silently if the path doesn't exist.

#### Fetch MCP

Lets Jan fetch and read any URL during a chat. No API key needed.
- Command: `/usr/local/bin/uvx`
- Args: `mcp-server-fetch`

#### Puppeteer MCP

Similar to Fetch but launches a full headless Chrome browser — useful for pages that require JavaScript to load. Heavier than Fetch and downloads Chromium (~170MB) on first run. **Only add this if Fetch can't handle a page you need.** For most web reading tasks, Fetch is sufficient.
- Command: `/usr/local/bin/npx`
- Args: `-y`, `@modelcontextprotocol/server-puppeteer`

> **Note:** Puppeteer downloads Chromium on first toggle and takes 1–2 minutes to become active. The spinner will run during this time — this is normal.

#### Adding Web Search

Click **+ Add MCP Server** in the top right. Two good options — pick one:

**Option A — Tavily Search** (recommended, 1000 searches/month free)
- Get a free API key at [app.tavily.com](https://app.tavily.com)
- Command: `/usr/local/bin/npx`
- Args: `-y tavily-mcp`
- Environment variable: `TAVILY_API_KEY` = your key

**Option B — Brave Search** (2000 searches/month free)
- Get a free API key at [brave.com/search/api](https://brave.com/search/api)
- Command: `/usr/local/bin/npx`
- Args: `-y @modelcontextprotocol/server-brave-search`
- Environment variable: `BRAVE_API_KEY` = your key

#### Jan Browser MCP (Built-in)

Jan ships a built-in "Jan Browser MCP" for browser control. To enable it:
1. Click `Install Extension →` in the MCP Servers page and install the Chrome extension
2. Set its Command to `/usr/local/bin/npx` (full path fix above)
3. Fully quit and restart Jan
4. Toggle it on — it will connect via the Chrome extension bridge

Works with any Chromium-based browser (Chrome, Ulaa, Arc, Brave, Edge). Does not require it to be your default browser — just have it open. Not compatible with Safari.

### Projects (v0.7.0+)

Jan supports **Projects** — workspaces that organise chats and files together. Create a Project (e.g., "School Work") and mount a local folder to give the AI persistent access to every file in that directory.

### Proactive Mode (v0.7.3+)

**Proactive Mode** allows Jan to "see" your active browser window to provide context without manual uploads. It is available under **Settings > Assistant**. Note: enabling this feature prompts Jan to download its own vision model (Jan-VL). If you do not want any models downloaded to the laptop, leave this off and use Open WebUI for vision tasks instead.

---

## 4. Vision & Image Tasks: Open WebUI

Because Jan cannot send images to a remote Ollama engine, **Open WebUI** is the recommended client for any task involving photos, screenshots, or documents with images. It connects directly to the Ollama server and supports full vision/multimodal capability with no local model downloads.

### Setup (run once per laptop)

**Option A — Docker:**
```bash
docker run -d -p 3000:3000 \
  -e OLLAMA_BASE_URL=http://resnet.local:11434 \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```
Then open `http://localhost:3000` in the browser.

**Option B — No install required:**
```bash
npx open-webui serve --ollama-url http://resnet.local:11434
```

Select `qwen3.5:9b` or `qwen3.5:27b` from the model picker. Drag-and-drop image upload works out of the box.

---

## 5. Intelligence Selection (RTX 3090 Optimized)

| Model | Ollama Tag | Capabilities | Speed |
| :--- | :--- | :--- | :--- |
| **Qwen 3.5 32B (VLLM)** | `qwen3.5:32b` | **Default.** Best instruction following + **Vision/Images**. | ~50 tok/s |
| **Qwen3-Coder 32B** | `qwen3-coder:32b` | **Logic.** Best for math, coding, and data organization. | ~50 tok/s |
| **Nemotron 30B** | `nemotron:30b` | **Chat.** Fine-tuned for natural, helpful conversation. | ~55 tok/s |

### Vision Tasks

For photos, screenshots, and image analysis, use **Open WebUI** (Section 4) with `qwen3.5:9b` or `qwen3.5:27b`. The GPU on the Linux Desktop does all the processing — nothing runs on the laptop.
