# 🚀 Getting Started

**[Telegram Builder (TG;BD)](https://github.com/AdriansaDev/TelegramBuilder/tree/main)** is a 100% open-source, self-hosted platform engineered in [Python](https://en.wikipedia.org/wiki/Python_(programming_language)) and [HTML](https://en.wikipedia.org/wiki/HTML) for architecting, deploying, and managing Telegram bots. Utilizing a visual, node-based layout, it abstracts complex backend logic into an intuitive graphical interface, enabling seamless event-driven workflow automation without requiring prior programming knowledge.

---


## 📖 Table of Contents

- [1. Getting Started](#1-getting-started)
  - [1.1. Overview & Index](#11-overview--index)
  - [1.2. Dependencies](#12-dependencies)
  - [1.3. Instalation Process](#13-instalation-process)
    - [Option A: Pre-compiled Binary (Recommended for Users)](#option-a-pre-compiled-binary-recommended-for-users)
    - [Option B: Source-based Setup (Recommended for Developers)](#option-b-source-based-setup-recommended-for-developers)
- [2. First Moves](#2-node-reference-manual)
  - [2.1: Overview & Index](#21-node-1-name)
  - [2.2. Primary Navigation](#22-node-2-name)
  - [2.3. Keyboard Shurtcuts](Documentation/DocumentationPortal.md#input-mapping-specification)

### 📦 Dependencies
To ensure seamless operation, the engine requires the following Python libraries:

| Library | Purpose |
| :--- | :--- |
| `pywebview` | Desktop UI orchestration and IPC bridging. |
| `python-telegram-bot` | Asynchronous Telegram API wrapper for bot logic. |
| `aiohttp` | Non-blocking HTTP client for external API integration. |

## 💻 Instalation Process

The platform provides two deployment methodologies depending on your operational requirements and host operating system.

### Option A: Pre-compiled Binaries (Standalone Deployment)
Designed for immediate execution without local runtime dependencies or python environment setups.

1. Navigate to the **[Releases](https://github.com/AdriansaDev/TelegramBuilder/releases)** section.
2. Download the architecture-specific compiled package for your operating system:
   * **Windows:** `TG-BD_vX.X_Windows.zip`
   * **Linux:** `TG-BD_vX.X_Linux.tar.gz`
   * **macOS:** `TG-BD_vX.X_macOS.dmg`
3. Extract the assets to an isolated directory and execute the standalone binary.

### Option B: Source-based Setup (Development Framework)
Designed for cross-platform environments requiring source auditing, debugging, or custom node expansion.

1. **Acquire the Source**
   Clone the repository topology and navigate into the root directory:
   ```bash
   git clone https://github.com/AdriansaDev/TelegramBuilder.git
   cd TelegramBuilder
   ```

2. **Environment Isolation**
   Initialize and activate a localized virtual environment to mitigate cross-platform dependency conflicts.

   * **Windows (Command Prompt):**
     ```cmd
     python -m venv venv
     .\venv\Scripts\activate.bat
     ```

   * **Windows (PowerShell):**
     ```powershell
     python -m venv venv
     .\venv\Scripts\Activate.ps1
     ```

   * **Linux / macOS (POSIX Shell):**
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```

3. **Dependency Resolution**
   Provision the required cross-platform runtime libraries via the package manager:
   ```bash
   pip install -r requirements.txt
   ```
   
4. **Initialization**
   Execute the primary lifecycle script to launch the multi-platform application instance:
   ```bash
   python main.py
   ```

---

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/%C2%AB_Home-Previous-blue?style=flat-square" alt="Previous"></a>
  <a href="./README.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./docs/PAGE_2.md"><img src="https://img.shields.io/badge/2-Active-blue?style=flat-square" alt="Page 2"></a>
  <a href="./docs/PAGE_3.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./docs/PAGE_2.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
