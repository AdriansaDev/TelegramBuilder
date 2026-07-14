# 🌀 TG;BD Documentation Portal

Welcome to the **Telegram Builder (TG;BD)** documentation — your comprehensive guide to building, deploying, and managing Telegram bots through a visual node-based interface.

---

## 📖 Table of Contents

| # | Document | Description |
|:---:|:---|:---|
| 1 | [Getting Started](GettingStarted.md) | Overview, dependencies, installation, first launch |
| 2 | [First Moves](FirstMoves.md) | Canvas navigation, input mapping, keyboard shortcuts, UI tour |
| 3 | [Node Reference](NodeReference.md) | Complete specification of all 31 node types |
| 4 | [Compiler Architecture](CompilerArchitecture.md) | How the JS compiler generates Python bot code |
| 5 | [API & IPC Reference](ApiIPC.md) | Backend Python API, IPC bridge, pywebview methods |
| 6 | [Advanced Workflows](AdvancedWorkflows.md) | Patterns, examples, and production deployment |
| 7 | [Troubleshooting](Troubleshooting.md) | Common errors, FAQ, debugging guide |

---

## 🧭 Quick Start

```bash
git clone https://github.com/AdriansaDev/TelegramBuilder.git
cd TelegramBuilder
pip install -r requirements.txt
python main.py
```

> **New here?** Start with [Getting Started](GettingStarted.md) for installation, then [First Moves](FirstMoves.md) to learn the canvas.

---

## 🏗️ Architecture Overview

```
┌────────────────────────────────────────────────────────┐
│                   Desktop Window (pywebview)            │
│  ┌──────────────────────────────────────────────────┐  │
│  │               editor.html (Chromium)              │  │
│  │  ┌──────────┐  ┌──────────────┐  ┌────────────┐  │  │
│  │  │ nodes.js │  │  compiler.js  │  │  Drawflow  │  │  │
│  │  │(Registry)│  │  (Generator)  │  │  (Canvas)  │  │  │
│  │  └──────────┘  └──────────────┘  └────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
│                        │ IPC Bridge                     │
│                        ▼                                │
│  ┌──────────────────────────────────────────────────┐  │
│  │               main.py (Python)                    │  │
│  │  ┌──────────────┐  ┌───────────────────────────┐ │  │
│  │  │ GraphCompiler │  │  Test Runner (subprocess) │ │  │
│  │  │  API          │  │  Log streaming, terminate │ │  │
│  │  └──────────────┘  └───────────────────────────┘ │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

---

## ✨ Key Features

- **Visual Node Graph** — Drag-and-drop bot construction with real-time connections
- **Self-Hosted Runtime** — Your token never leaves your machine
- **Live Testing** — Deploy and test bots directly from the editor
- **31 Node Types** — Triggers, actions, logic, variables, and system operations
- **Inline Keyboard Builder** — Visual button row/column editor
- **Session Isolation** — Per-chat state management out of the box
- **Cross-Platform** — Windows, Linux, macOS via pywebview

---

<p align="center">
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/Start_Here-%C2%BB_Getting_Started-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
