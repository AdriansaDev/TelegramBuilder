# 🚀 Getting Started

**[Telegram Builder (TG;BD)](https://github.com/AdriansaDev/TelegramBuilder)** is a 100% open-source, self-hosted platform for architecting, deploying, and managing Telegram bots through a visual node-based interface. It abstracts complex backend logic into an intuitive graphical workflow — no prior programming knowledge required.

---

## 📖 Table of Contents

- [1. Overview & Index](#1-overview--index)
- [2. Dependencies](#2-dependencies)
- [3. Installation Process](#3-installation-process)
  - [Option A: Pre-compiled Binary (Recommended for Users)](#option-a-pre-compiled-binary-recommended-for-users)
  - [Option B: Source-based Setup (Recommended for Developers)](#option-b-source-based-setup-recommended-for-developers)
- [4. First Launch](#4-first-launch)
- [5. Generating a Bot Token](#5-generating-a-bot-token)

---

## 1. Overview & Index

### How It Works

| Layer | Technology | Role |
|:---|:---|:---|
| **Desktop Shell** | [pywebview](https://pywebview.flowrl.com/) | Native OS window with embedded Chromium |
| **Frontend** | HTML + CSS + JavaScript (Vanilla) | Drawflow canvas, node registry, compiler |
| **Backend** | Python 3.8+ | IPC bridge, test subprocess management, clipboard |
| **Bot Runtime** | [python-telegram-bot v21.x](https://python-telegram-bot.org/) | Async Telegram API wrapper |
| **HTTP Client** | [aiohttp](https://docs.aiohttp.org/) | Non-blocking HTTP requests from generated bots |

### Flow of a Bot Build

```
1. Launch TG;BD → 2. Drag nodes onto canvas → 3. Configure node properties
→ 4. Connect nodes with wires → 5. Click "Compile & Download" → 6. ???
→ 7. Your bot is running on Telegram!
```

---

## 2. Dependencies

| Library | Version | Purpose |
|:---|:---|:---|
| `pywebview` | >=5.3 | Desktop UI orchestration and IPC bridging |
| `python-telegram-bot` | >=21.10 | Asynchronous Telegram API wrapper |
| `aiohttp` | >=3.11 | Non-blocking HTTP client for external API integration |

> **Zero JavaScript dependencies.** The frontend uses vanilla JS, [Drawflow](https://github.com/jerosoler/Drawflow) (embedded), and [GSAP](https://greensock.com/gsap/) (loaded via CDN).

---

## 3. Installation Process

### Option A: Pre-compiled Binary (Recommended for Users)

Designed for immediate execution without Python or dependency management.

1. Navigate to the **[Releases](https://github.com/AdriansaDev/TelegramBuilder/releases)** page.
2. Download the package for your operating system:

| OS | Package |
|:---|:---|
| **Windows** | `TG-BD_vX.X_Windows.zip` |
| **Linux** | `TG-BD_vX.X_Linux.tar.gz` |
| **macOS** | `TG-BD_vX.X_macOS.dmg` |

3. Extract to an isolated directory and run the executable.

---

### Option B: Source-based Setup (Recommended for Developers)

#### Step 1: Clone the Repository

```bash
git clone https://github.com/AdriansaDev/TelegramBuilder.git
cd TelegramBuilder
```

#### Step 2: Create a Virtual Environment

Isolate dependencies to avoid conflicts with other Python projects.

<details>
<summary><b>Windows (Command Prompt)</b></summary>

```cmd
python -m venv venv
.\venv\Scripts\activate.bat
```
</details>

<details>
<summary><b>Windows (PowerShell)</b></summary>

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```
</details>

<details>
<summary><b>Linux / macOS</b></summary>

```bash
python3 -m venv venv
source venv/bin/activate
```
</details>

#### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

#### Step 4: Launch

```bash
python main.py
```

> The application window will open with the Drawflow canvas ready for bot construction.

---

## 4. First Launch

When you run TG;BD for the first time, you'll see:

<img width="1920" height="1013" alt="image" src="https://github.com/user-attachments/assets/330c8aee-4cb8-4fcf-bc80-8a355a8a1e1b" />

### What You See

| Element | Location | Purpose |
|:---|:---|:---|
| **Navbar** | Top (64px fixed) | Logo, action buttons (Clear, Live Test, Compile) |
| **Canvas** | Center | Infinite grid workspace for node placement |
| **Zoom Controls** | Bottom-right | Zoom in/out, manual scale input |
| **Context Menu** | Right-click on canvas | Node creation menu with search |

### What to Do Next

- **Right-click** on the canvas to open the node menu
- Select a **Trigger** node (e.g., "Command Trigger")
- Connect it to an **Action** node (e.g., "Send Message")
- Configure the message text and your bot token
- Click **Compile & Download** to generate the Python script

---

## 5. Generating a Bot Token

To test or deploy your bot, you need a **Telegram Bot Token**:

1. Open Telegram and search for [@BotFather](https://t.me/BotFather)
2. Send `/newbot` and follow the prompts
3. BotFather will give you a token like: `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`

> **Keep your token secret.** Anyone with your token can control your bot. TG;BD never transmits your token over the network — it stays local.

---

## ✅ Installation Checklist

- [ ] I have Python 3.8+ installed
- [ ] Virtual environment created and activated
- [ ] All pip dependencies installed
- [ ] `python main.py` launches the window
- [ ] I have a Telegram bot token from BotFather
- [ ] I can right-click and see the node creation menu

---

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/%C2%AB_Home-blue?style=flat-square" alt="Home"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/Next-%C2%BB_First_Moves-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Active-blue?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
