# 🌀 Telegram Builder (TG;BD)

**Low-code Telegram Bot Framework** — Architect robust, scalable bots through a visual node-based interface. No coding required.

<p align="center">
  <img width="1920" height="1013" alt="image" src="https://github.com/user-attachments/assets/1bd97c9b-64c8-45ff-acbf-bae711c37abb" />
</p>

<p align="center">
  <a href="#-features"><img src="https://img.shields.io/badge/Features-8A2BE2?style=flat-square" alt="Features"></a>
  <a href="#-quick-start"><img src="https://img.shields.io/badge/Quick_Start-00BFFF?style=flat-square" alt="Quick Start"></a>
  <a href="#-documentation"><img src="https://img.shields.io/badge/Documentation-228B22?style=flat-square" alt="Documentation"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow?style=flat-square" alt="License"></a>
  <a href="https://github.com/AdriansaDev/TelegramBuilder/releases"><img src="https://img.shields.io/badge/Download-Latest-blue?style=flat-square" alt="Download"></a>
</p>

---

## ✨ Features

| Capability | Description |
|:---|:---|
| **Visual Node Graph** | Drag-and-drop bot construction with real-time wiring |
| **31 Node Types** | Triggers, actions, logic, variables, and system operations |
| **Self-Hosted** | Your bot token never leaves your machine |
| **Live Testing** | Deploy and test bots directly from the editor |
| **Visual Keyboard Builder** | Inline button rows with automatic callback wiring |
| **Session Isolation** | Per-chat state management out of the box |
| **Cross-Platform** | Windows, Linux, macOS via pywebview |
| **Zero JS Dependencies** | Vanilla JavaScript, embedded Drawflow + GSAP via CDN |

---

## 🚀 Quick Start

### Option A: Binary (Recommended)

1. Download from [Releases](https://github.com/AdriansaDev/TelegramBuilder/releases)
2. Extract and run

### Option B: Source

```bash
git clone https://github.com/AdriansaDev/TelegramBuilder.git
cd TelegramBuilder
python -m venv venv

# Windows: venv\Scripts\activate
# Linux/macOS: source venv/bin/activate

pip install -r requirements.txt
python main.py
```

### Get a Bot Token

1. Open Telegram, search for [@BotFather](https://t.me/BotFather)
2. Send `/newbot` and follow the prompts
3. Use the token when prompted in TG;BD

---

## 📚 Documentation

| Document | Description |
|:---|:---|
| [Documentation Portal](Documentation/DocumentationPortal.md) | Main hub with architecture diagram and full TOC |
| [Getting Started](Documentation/GettingStarted.md) | Installation, dependencies, first launch |
| [First Moves](Documentation/FirstMoves.md) | Canvas navigation, input mapping, keyboard shortcuts |
| [Node Reference](Documentation/NodeReference.md) | Complete specification of all 31 node types |
| [Compiler Architecture](Documentation/CompilerArchitecture.md) | How the compiler generates Python bot code |
| [API & IPC Reference](Documentation/ApiIPC.md) | Backend API and IPC bridge methods |
| [Advanced Workflows](Documentation/AdvancedWorkflows.md) | Production patterns and deployment |
| [Troubleshooting](Documentation/Troubleshooting.md) | Common errors, FAQ, debugging guide |

---

## 🏗️ Architecture

```
┌────────────────────────────────────────────────────┐
│                   Desktop Window (pywebview)         │
│  ┌──────────────────────────────────────────────┐  │
│  │               editor.html (Chromium)          │  │
│  │  ┌──────────┐  ┌──────────┐  ┌────────────┐ │  │
│  │  │ nodes.js │  │compiler.js│  │  Drawflow  │ │  │
│  │  │(Registry)│  │(Generator)│  │  (Canvas)  │ │  │
│  │  └──────────┘  └──────────┘  └────────────┘ │  │
│  └──────────────────────────────────────────────┘  │
│                         │ IPC                       │
│  ┌──────────────────────────────────────────────┐  │
│  │               main.py (Python)                │  │
│  │  ┌──────────────┐  ┌──────────────────────┐  │  │
│  │  │ GraphCompiler│  │  Test Runner (proc)   │  │  │
│  │  │     API       │  │  Log stream, kill    │  │  │
│  │  └──────────────┘  └──────────────────────┘  │  │
│  └──────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────┘
```

---

## 🔧 Requirements

| Dependency | Version |
|:---|:---|
| Python | 3.8+ |
| pywebview | >= 5.3 |
| python-telegram-bot | >= 21.10 |
| aiohttp | >= 3.11 |

---

## 🗺️ Roadmap

- [x] Visual node graph with Drawflow
- [x] 31 node types across 6 categories
- [x] Live test subprocess runner
- [x] Inline keyboard builder
- [x] Error handler auto-pairing
- [ ] Multi-bot dashboard management
- [ ] Bots / Commands marketplace
- [ ] Custom node SDK
- [ ] Web-based deployment (VPS/cloud)

---

## ⚖️ License

This project is licensed under the **MIT License**.

- **Permissions:** Free to use, modify, redistribute, and commercially utilize
- **Attribution:** Credit required to original author (**AdriansaDev**)
- **No Warranty:** Provided "as is", without warranty of any kind

See [LICENSE](LICENSE) for full details.

---

<p align="center">
  <a href="Documentation/DocumentationPortal.md"><b>📖 Read the Full Documentation</b></a>
</p>
