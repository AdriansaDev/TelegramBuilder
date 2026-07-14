# 🔧 Troubleshooting

Common issues, error messages, and solutions for Telegram Builder.

---

## 📖 Table of Contents

- [1. Installation Issues](#1-installation-issues)
- [2. Compiler Errors](#2-compiler-errors)
- [3. Live Test Errors](#3-live-test-errors)
- [4. Runtime Errors (Generated Bot)](#4-runtime-errors-generated-bot)
- [5. UI & Canvas Issues](#5-ui--canvas-issues)
- [6. Platform-Specific Issues](#6-platform-specific-issues)
- [7. FAQ](#7-faq)

---

## 1. Installation Issues

### `pip install -r requirements.txt` fails

| Symptom | Likely Cause | Solution |
|:---|:---|:---|
| `pywebview` fails on Linux | Missing GTK/GNOME dependencies | `sudo apt install libgtk-3-dev libwebkit2gtk-4.0-dev` |
| `python-telegram-bot` fails | Python < 3.8 | `python --version` — upgrade to 3.8+ |
| `pywebview` fails on macOS | Missing PyObjC | `pip install pyobjc-framework-Cocoa` |
| Permission denied | Virtual env not activated | Activate venv first, then `pip install` |

### Window doesn't open

```
python main.py
# ... no output, no window
```

| Check | Command |
|:---|:---|
| Python version | `python --version` (need 3.8+) |
| Dependencies | `pip list \| findstr pywebview` (Windows) |
| Firewall | Allow Python in firewall |
| GPU drivers | Update graphics drivers (especially on Linux) |

---

## 2. Compiler Errors

### "IndentationError: expected an indented block"

```
File "tmpXXXX.py", line 29
    except Exception as e:
    ^
IndentationError: expected an indented block after 'try' statement on line 28
```

**Cause:** A trigger node is connected to a node that doesn't exist in the graph (stale connection from a deleted node). The compiler emits `try:` but `traverse()` finds no valid node and generates no body code.

**Fix:**
1. Check the trigger's connections — delete and re-create any suspect wires
2. Ensure the Error Handler is properly connected
3. If the node exists but is disconnected, reconnect it
4. Re-compile after verifying all connections

### Generated script has `pass` in unexpected places

**Cause:** Some nodes in the chain reference non-existent graph entries (orphan connections).

**Fix:** Delete the orphaned connections and re-wire the affected nodes.

### "Unexpected token" in compiler.js

**Cause:** Modified `nodes.js` or `compiler.js` with syntax errors.

**Fix:** Check the file for unmatched braces, missing commas, or invalid JavaScript:
```bash
# Check brace balance
$c = Get-Content compiler.js -Raw
$o = ($c.ToCharArray() | Where-Object { $_ -eq '{' }).Count
$cl = ($c.ToCharArray() | Where-Object { $_ -eq '}' }).Count
Write-Host "Open: $o, Close: $cl (balanced: $($o -eq $cl))"
```

---

## 3. Live Test Errors

### Test doesn't start

```
[BRIDGE] Subprocess started with PID 0
```

**Causes:**
| Issue | Solution |
|:---|:---|
| No token configured | Enter your bot token in the Live Test modal |
| Token is invalid | Check token format: `123456:ABCdef` |
| Subprocess error | Check the console output for Python tracebacks |
| Temp file permission | Run as administrator (Windows) or check `/tmp` permissions |

### Test starts but immediately stops

```
Process exited with code 1
```

**Solutions:**
1. Check the console output for the actual error
2. Test the generated script manually: `python temp_script.py`
3. Ensure all dependencies are installed in the host Python environment
4. Look for syntax errors in the generated code

### No logs appearing

**Causes:**
| Issue | Solution |
|:---|:---|
| Buffered output | Add `flush=True` or `sys.stdout.flush()` to the bot code |
| Subprocess not started | Check `[BRIDGE]` message for actual PID |
| Poll interval too slow | Default is 400ms — check if intervals are running |

---

## 4. Runtime Errors (Generated Bot)

### `AttributeError: 'NoneType' object has no attribute 'text'`

**Cause:** The bot tried to access `message.text` when `message` is `None` (e.g., on non-text updates like a sticker in a text-only trigger).

**Solutions:**
1. Use the **Any Message Trigger** with appropriate content type checkboxes
2. Add a `logic_if` node to check `message.text` before using it
3. Make sure the trigger's content type filter includes the type of incoming update

### `telegram.error.Forbidden: Bot was kicked from the chat`

**Cause:** The bot was removed from the chat but code still references it.

**Solutions:**
1. Re-add the bot to the chat
2. Check `logic_if` conditions before admin actions
3. Add try/except around sensitive operations

### `ChatPermissions` import error

```python
NameError: name 'ChatPermissions' is not defined
```

**Cause:** Very old generated scripts that don't include the import. If you're using a current version, this should not happen.

**Fix:** Regenerate the script with the latest compiler. The `ChatPermissions` import is at the top of every generated file.

### State variables not persisting

**Cause:** Each chat gets its own state dictionary. Variables set in one handler may not be available in another if they use different chat IDs.

**Solutions:**
1. Verify `chat_id` is consistent across handlers
2. Use `chat_states.setdefault('global', {})` for cross-chat data (startup context)
3. Check that the trigger type provides `update.effective_chat.id`

### Button callbacks not working

| Symptom | Cause | Fix |
|:---|:---|:---|
| Button press does nothing | Callback data mismatch | Check `callback_data` in keyboard builder matches handler pattern |
| "Query is too old" | Timeout | Process callbacks within 30 seconds |
| Wrong handler fires | Duplicate callback patterns | Ensure each button has a unique callback identifier |

### Bot doesn't respond to commands

| Check | Command/Test |
|:---|:---|
| Token correct? | `curl https://api.telegram.org/bot<TOKEN>/getMe` |
| Bot added to group? | Add bot as admin (for admin features) |
| Command registered? | Check `add_handler(CommandHandler(...))` in generated code |
| Filter conflicts? | MessageHandler + CommandHandler order matters — commands first |

---

## 5. UI & Canvas Issues

### Context menu doesn't open

| Cause | Solution |
|:---|:---|
| Clicked on a node | Right-click on empty canvas space |
| Event propagation blocked | Reload the app (Ctrl+R won't work; restart pywebview) |
| GSAP not loaded | Check internet connection (GSAP is loaded via CDN) |

### Nodes won't connect

| Cause | Solution |
|:---|:---|
| Input already has a connection | Disconnect the existing wire first |
| Wrong output-input direction | Drag from output (right) to input (left) |
| Connection would create a loop | Drawflow prevents circular connections |

### Zoom doesn't work

| Cause | Solution |
|:---|:---|
| Zoom at limit (0.10x or 3.00x) | Buttons won't go past limits. Use manual input for extreme values |
| CSS conflict | Reload the application |
| Keyboard shortcut conflict | Check other applications using Ctrl+Wheel |

### Canvas is empty / nodes disappeared

1. Check `localStorage` — the canvas autosaves
2. Open browser dev tools (if in browser mode)
3. Check `editor.drawflow.Home.data` in console
4. If corrupted, click "Clear Canvas" to reset

---

## 6. Platform-Specific Issues

### Windows

| Issue | Solution |
|:---|:---|
| Antivirus blocks temp script | Add project folder to antivirus exclusions |
| pywebview black screen | Update GPU drivers, install WebView2 runtime |
| `clip` command fails | Ensure `clip.exe` is in PATH (should be by default on Windows 10+) |
| Unicode in console | Set console to UTF-8: `chcp 65001` |

### Linux

| Issue | Solution |
|:---|:---|
| `xclip` not installed | `sudo apt install xclip` |
| No window decoration | Install `gnome-shell` or `unity` |
| GTK assertion errors | `sudo apt install libgtk-3-dev libwebkit2gtk-4.0-dev` |
| Wayland issues | Run under X11: `env QT_QPA_PLATFORM=xcb python main.py` |

### macOS

| Issue | Solution |
|:---|:---|
| `pbcopy` permission | Grant Terminal/APP access in System Preferences → Security → Privacy → Accessibility |
| Gatekeeper blocks pywebview | Allow in System Preferences → Security & Privacy → General |
| Python not found | Use `python3` instead of `python` |

---

## 7. FAQ

### Can I use this on a server without a display?

Yes. TG;BD is designed to build bots visually, but the generated `.py` file runs anywhere with Python. You can:
- Build on your desktop
- Transfer the `.py` file to your server
- Run it with `python tg_bot_build.py`

### Can I modify the generated code?

Absolutely. The generated `.py` file is standard Python. You can edit it directly — add new handlers, modify logic, or integrate with other Python libraries.

### Is my bot token safe?

Yes. TG;BD is fully self-hosted:
- Your token never leaves your machine
- No telemetry, no analytics, no cloud dependencies
- The token is embedded in the generated script, which you control

### Can I contribute new node types?

Yes! See the [Compiler Architecture](./CompilerArchitecture.md#-extending-the-compiler) guide for the extension protocol. The node system is designed to be extensible.

### Why does the window show a white screen?

| Cause | Solution |
|:---|:---|
| pywebview initialization | Wait a few seconds — first load can be slow |
| GPU driver issue | Update graphics drivers |
| Missing WebView2 (Windows) | Install Microsoft Edge WebView2 Runtime |
| Missing WebKit (Linux) | `sudo apt install webkit2gtk-4.0` |

### How do I reset the canvas?

Click the **Clear Canvas** button in the navbar, then confirm "Purge Workspace". This removes all nodes, connections, and resets the Undo/Redo history.

### Why are my variables empty in the next handler?

State is per-chat. If the user changes chat (or the bot is used in a different group), their state starts fresh. Use `trigger_startup` with `var_define` to set defaults.

### How do I add a delay in milliseconds?

Set `unit` to `milliseconds` in the **Delay (Wait)** node's configuration. The compiler multiplies by 0.001 internally.

### Can I host multiple bots?

Yes. Each generated `.py` file is a standalone bot. Run them as separate processes or in Docker containers.

---

## 📋 Quick Diagnosis Checklist

- [ ] Python 3.8+
- [ ] All pip dependencies installed
- [ ] Bot token is valid
- [ ] Trigger node is connected (wire visible)
- [ ] Error Handler is connected
- [ ] Token entered in Live Test / Compile modal
- [ ] Generated script is syntactically valid Python
- [ ] No orphaned connections (wires to deleted nodes)

---

<p align="center">
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/%C2%AB_Previous-Advanced_Workflows-blue?style=flat-square" alt="Previous"></a>
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/%C2%AB_Home-blue?style=flat-square" alt="Home"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Active-blue?style=flat-square" alt="Page 7"></a>
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Home-%C2%AB-blue?style=flat-square" alt="Home"></a>
</p>
