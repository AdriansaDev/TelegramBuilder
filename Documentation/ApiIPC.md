# 🔌 API & IPC Reference

**Telegram Builder (TG;BD)** uses a synchronous IPC bridge between the JavaScript frontend (Chromium) and the Python backend via [pywebview](https://pywebview.flowrl.com/). This document describes every API method, the IPC protocol, and how to extend the backend.

---

## 📖 Table of Contents

- [1. IPC Architecture](#1-ipc-architecture)
- [2. JavaScript → Python API](#2-javascript--python-api)
  - [run_test](#run_test)
  - [terminate_test](#terminate_test)
  - [get_logs](#get_logs)
  - [is_test_running](#is_test_running)
  - [download_script](#download_script)
  - [copy_to_clipboard](#copy_to_clipboard)
- [3. Python Backend (`main.py`)](#3-python-backend-mainpy)
- [4. Browser Fallback](#4-browser-fallback)
- [5. Extending the API](#5-extending-the-api)

---

## 1. IPC Architecture

```
┌────────────────────────────────────────────────────────┐
│                  JavaScript (editor.html)               │
│                                                        │
│  window.pywebview.api.method(args)                      │
│       │                                                 │
│       │  pywebview IPC bridge                           │
│       ▼                                                 │
│  Python GraphCompilerAPI instance                       │
│       │                                                 │
│       ├── run_test(script)       → spawns subprocess    │
│       ├── terminate_test()       → kills subprocess     │
│       ├── get_logs()             → drains log queue     │
│       ├── is_test_running()      → checks process       │
│       ├── download_script(script) → Save File dialog    │
│       └── copy_to_clipboard(script) → platform clipboard│
└────────────────────────────────────────────────────────┘
```

### Bridge Protocol

| Property | Value |
|:---|:---|
| **Transport** | In-process Chromium-Python bridge (pywebview) |
| **Method Style** | Synchronous call, return value |
| **Error Handling** | Exceptions propagate to JavaScript as rejected promises |
| **Threading** | `run_test` spawns a daemon thread for log reading |
| **Subprocess** | Test scripts run in a separate Python process |

### Thread Safety

| Resource | Thread Safe? | Mechanism |
|:---|:---|:---|
| `self.log_queue` (Queue) | ✅ Yes | `queue.Queue` is thread-safe |
| `self.active_process` (Popen) | ✅ Yes | Only accessed from main thread |
| File operations | ✅ Yes | Temp files unique per call |
| Clipboard (subprocess) | ⚠️ Platform-dependent | Sequential, one-shot calls |

---

## 2. JavaScript → Python API

### `run_test`

Invokes the Live Test subprocess with a compiled Python script.

```javascript
const result = await window.pywebview.api.run_test(script);
```

| Parameter | Type | Description |
|:---|:---|:---|
| `script` | String | Complete Python bot script (output of `compiler.compile()`) |

| Return | Type | Description |
|:---|:---|:---|
| Success | String | `"Subprocess started with PID {pid}"` |
| Failure | Error | Propagates exception (e.g., invalid Python path) |

**Behavior:**
1. Terminates any previously running test process
2. Writes `script` to a temporary `.py` file (UTF-8)
3. Spawns: `subprocess.Popen([sys.executable, tmp_path], stdout=PIPE, stderr=PIPE, text=True, bufsize=1)`
4. Starts a daemon thread reading stdout/stderr line-by-line into a `Queue`
5. Returns immediately with the PID

**Frontend usage (editor.html):**
```javascript
async function launchTestScript(script) {
    const result = await window.pywebview.api.run_test(script);
    const pid = result.match(/PID (\d+)/)[1];
    showNotification(`Test running (PID: ${pid})`, 'success');
    startLogPolling();
}
```

---

### `terminate_test`

Terminates the currently running test subprocess.

```javascript
await window.pywebview.api.terminate_test();
```

| Parameters | None |
|:---|:---|
| Return | `undefined` |

**Termination Strategy:**

| OS | Primary | Fallback |
|:---|:---|:---|
| **Windows** | `taskkill /F /T /PID {pid}` (force-kills process tree) | — |
| **macOS / Linux** | `process.terminate()` (SIGTERM) | 3s timeout → `process.kill()` (SIGKILL) |

After termination:
- Waits for log reader thread to join (3s timeout)
- Clears remaining log entries from queue
- Sets `active_process = None`

**Frontend usage:**
```javascript
async function terminateTest() {
    if (window.pywebview) {
        await window.pywebview.api.terminate_test();
    }
    testRunning = false;
    hideTestModal();
}
```

---

### `get_logs`

Retrieves accumulated log output from the running test subprocess.

```javascript
const logs = await window.pywebview.api.get_logs();
```

| Parameters | None |
|:---|:---|
| Return | `String` or `null` |

| Return Value | Meaning |
|:---|:---|
| `"line1\nline2\n..."` | New log lines since last poll |
| `null` | No new output |

**Polling behavior (frontend):**
```javascript
// Poll every 400ms while test is running
const pollInterval = setInterval(async () => {
    const logs = await window.pywebview.api.get_logs();
    if (logs) {
        const output = document.getElementById('console-output');
        output.textContent += logs;
        output.scrollTop = output.scrollHeight;
    }
    const stillRunning = await window.pywebview.api.is_test_running();
    if (!stillRunning) clearInterval(pollInterval);
}, 400);
```

---

### `is_test_running`

Checks whether the test subprocess is still active.

```javascript
const running = await window.pywebview.api.is_test_running();
```

| Parameters | None |
|:---|:---|
| Return | `Boolean` |

Returns `true` if `active_process` is not `None` and `poll()` returns `None` (process still running).

---

### `download_script`

Opens a native **Save File** dialog and writes the compiled script to disk.

```javascript
await window.pywebview.api.download_script(script);
```

| Parameter | Type | Description |
|:---|:---|:---|
| `script` | String | Complete Python bot script |

| Return | Type | Description |
|:---|:---|:---|
| Success | `undefined` | File saved at user-selected path |
| Cancel | `undefined` | User closed dialog without saving |

**Dialog configuration:**
- Default filename: `tg_bot_build.py`
- File type filter: `Python Files (*.py)`

---

### `copy_to_clipboard`

Copies the compiled script to the system clipboard using the platform-native clipboard tool.

```javascript
await window.pywebview.api.copy_to_clipboard(script);
```

| Parameter | Type | Description |
|:---|:---|:---|
| `script` | String | Complete Python bot script |

| Platform | Tool Used | Command |
|:---|:---|:---|
| **Windows** | `clip` | `echo script_text \| clip` (via `subprocess.run(['clip'], input=script, text=True)`) |
| **macOS** | `pbcopy` | `subprocess.run(['pbcopy'], input=script, text=True)` |
| **Linux** | `xclip` | `subprocess.run(['xclip', '-selection', 'clipboard'], input=script, text=True)` |

> **Security note:** Uses command lists (not `shell=True`), preventing shell injection.

---

## 3. Python Backend (`main.py`)

### Class: `GraphCompilerAPI`

```python
class GraphCompilerAPI:
    def __init__(self):
        self.active_process = None
        self.log_queue = Queue()
```

### Method Signatures

```python
def run_test(self, script: str) -> str:
    """Spawn subprocess running the compiled script."""

def terminate_test(self) -> None:
    """Kill the active subprocess and clean up."""

def get_logs(self) -> Union[str, None]:
    """Drain the log queue and return accumulated lines."""

def is_test_running(self) -> bool:
    """Check if subprocess is still alive."""

def download_script(self, script: str) -> None:
    """Open Save File dialog and write script to disk."""

def copy_to_clipboard(self, script: str) -> None:
    """Copy script text to system clipboard."""
```

### Application Entry Point

```python
if __name__ == '__main__':
    api = GraphCompilerAPI()
    window = webview.create_window(
        'TG;BD | Telegram Bot Builder',
        'editor.html',
        js_api=api,
        width=1280,
        height=800,
        resizable=True
    )
    webview.start()
```

| `create_window` Parameter | Value | Description |
|:---|:---|:---|
| `title` | `'TG;BD \| Telegram Bot Builder'` | Window title |
| `url` | `'editor.html'` | Frontend HTML file |
| `js_api` | `api` | Python object exposed to JavaScript |
| `width` | `1280` | Initial window width |
| `height` | `800` | Initial window height |
| `resizable` | `True` | Allow window resize |

---

## 4. Browser Fallback

The frontend detects whether it's running inside pywebview:

```javascript
if (window.pywebview) {
    // Call IPC methods
    await window.pywebview.api.run_test(script);
} else {
    // Running in a regular browser
    console.warn('pywebview not available');
}
```

### Behavior in Browser Mode

| Feature | Browser | pywebview |
|:---|:---|:---|
| Canvas & Node editing | ✅ Full | ✅ Full |
| Compile & Preview | ✅ (copy manually) | ✅ (copy/download) |
| Live Test | ❌ Disabled | ✅ Full |
| Download Script | ❌ Disabled | ✅ Native dialog |
| Clipboard | ❌ Disabled | ✅ Platform-native |

> **Browser mode** is useful for UI development/testing. The HTML can be served via any static file server. Live test and file download require the full pywebview environment.

---

## 5. Extending the API

### Adding a New Backend Method

1. **Add the method to `GraphCompilerAPI`** in `main.py`:
   ```python
   def my_new_method(self, arg1: str, arg2: int) -> dict:
       result = do_something(arg1, arg2)
       return {"status": "ok", "data": result}
   ```

2. **Call from JavaScript** in `editor.html`:
   ```javascript
   const response = await window.pywebview.api.my_new_method("hello", 42);
   ```

### Rules for IPC Methods

| Rule | Reason |
|:---|:---|
| Must be synchronous | pywebview JS API returns a promise that resolves to the return value |
| Return JSON-serializable types | `str`, `int`, `float`, `bool`, `list`, `dict`, `None` |
| Avoid long-running operations | Blocks the UI thread. Use threading + polling for long tasks |
| Use `queue.Queue` for streaming | Thread-safe log output delivery |
| Never use `shell=True` | Prevents shell injection in subprocess calls |

### Current Thread Model

```
Main Thread (UI)
    │
    ├── run_test()        → spawns subprocess + daemon thread
    ├── terminate_test()  → kills subprocess + joins thread
    ├── get_logs()        → reads from queue
    ├── is_test_running() → checks process
    ├── download_script() → file dialog (blocking)
    └── copy_to_clipboard() → subprocess call (blocking)

Daemon Thread
    │
    └── Reads stdout/stderr lines → pushes to log_queue
```

---

<p align="center">
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/%C2%AB_Previous-Compiler_Architecture-blue?style=flat-square" alt="Previous"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/Next-%C2%BB_Advanced_Workflows-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Active-blue?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
