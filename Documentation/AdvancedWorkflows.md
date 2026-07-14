# 🧠 Advanced Workflows

Production patterns, real-world examples, and optimization strategies for building sophisticated Telegram bots with TG;BD.

---

## 📖 Table of Contents

- [1. Pattern: Welcome & Rules Bot](#1-pattern-welcome--rules-bot)
- [2. Pattern: Admin Command Dashboard](#2-pattern-admin-command-dashboard)
- [3. Pattern: Feedback Collector](#3-pattern-feedback-collector)
- [4. Pattern: Scheduled Notifications (Startup)](#4-pattern-scheduled-notifications-startup)
- [5. Variable Interpolation Guide](#5-variable-interpolation-guide)
- [6. Error Handling Best Practices](#6-error-handling-best-practices)
- [7. Performance Optimization](#7-performance-optimization)
- [8. Production Deployment](#8-production-deployment)

---

## 1. Pattern: Welcome & Rules Bot

A bot that welcomes new members, presents rules, and requires acceptance.

### Node Graph

```
[New Chat Member Trigger]
         │
         ▼
[Send Message: "Welcome! Please read the rules:"]
  ├── Button: "📜 View Rules" (cb: view_rules)
  └── Button: "✅ Accept" (cb: accept_rules)
         │
         ├── view_rules ──► [Send Message: "1. Be nice... 2. ..."]
         │
         └── accept_rules ──► [Check Variable: has_accepted]
                                  │
                             Yes  │  No
                                  ▼
                          [Set Description: "+1 member"]
                                  │
                                  ▼
                          [Define Variable: has_accepted = True]
                                  │
                                  ▼
                          [Send Message: "🎉 Welcome aboard!"]
```

### Configuration Details

| Node | Config |
|:---|:---|
| **Trigger** | `trigger_new_member` (no config) |
| **Send Message** | Text: `Welcome {{user}}! Please read the rules:` with keyboard |
| **Check Variable** | Key: `has_accepted`, Value: `True` |
| **Define Variable** | Key: `has_accepted`, Type: `Boolean`, Value: `true` |

### Generated Code Highlights

```python
async def handler_123(update, context):
    try:
        chat_id = update.effective_chat.id if update.effective_chat else None
        state = chat_states.setdefault(chat_id, {}) if chat_id else {}
        message = SafeMessage(update.message, update.effective_user) if update.message else None

        markup_456 = InlineKeyboardMarkup([
            [InlineKeyboardButton("📜 View Rules", callback_data="view_rules"),
             InlineKeyboardButton("✅ Accept", callback_data="accept_rules")],
        ])
        eval_text = f"""Welcome! Please read the rules:"""
        sent_msg = await context.bot.send_message(chat_id=message.chat.id, text=eval_text, parse_mode='Markdown', reply_markup=markup_456)
        state['__last_bot_message_id'] = sent_msg.message_id
        state['__last_bot_chat_id'] = sent_msg.chat.id
    except Exception as e:
        logging.error(f"Error in handler_123: {e}")
```

---

## 2. Pattern: Admin Command Dashboard

A multi-command admin panel using `trigger_cmd` nodes and `logic_if` branching.

### Node Graph

```
[/admin Command Trigger]  ──► [If Condition: message.from_user.id == ADMIN_ID]
                                       │
                                  True  │  False
                                       ▼
                                 [If Condition: message.text == '/admin stats']
                                    True │  False
                                         ▼
                              [Send Message: "Stats: ..."]
                                    
                         False path ──► [If Condition: message.text == '/admin broadcast']
                                           True │  False
                                                ▼
                                       [Send Message: "Broadcast text:"]
                                                │
                                                ▼
                                       [Wait for next message...]
```

### Implementation Notes

| Challenge | Solution |
|:---|:---|
| Admin ID hardcoding | Use `var_define` at startup with your Telegram user ID |
| Multi-level menus | Chain `logic_if` nodes for each command |
| Persistent state | Use `var_define` at startup to set `is_admin = False`, then update on `/admin` |

### Security Note

Always check `message.from_user.id` against a stored admin ID. Never rely on `message.chat.id` alone, as anyone in the chat could trigger the command.

---

## 3. Pattern: Feedback Collector

Collect user feedback with a multi-step conversation.

### Node Graph

```
[Command Trigger: /feedback]
         │
         ▼
[Send Message: "Please enter your feedback:"]
         │
         ▼
[Wait...] (Wait for next user message)
         │
         ▼
[HTTP Request: POST to API]
         │
         ▼
[Send Message: "Thank you! Feedback received."]
```

### State Management

The feedback collector uses state variables to track conversation stage:

```python
# Step 1: User triggers /feedback
state['feedback_stage'] = 'awaiting_feedback'

# Step 2: Next message detection (via logic_if)
if state.get('feedback_stage') == 'awaiting_feedback':
    state['feedback_text'] = message.text
    state['feedback_stage'] = 'complete'
```

### HTTP Integration

Configure the `sys_http` node:

| Config Field | Value |
|:---|:---|
| `method` | `POST` |
| `url` | `https://api.example.com/feedback` |
| `save_var` | `feedback_result` |

---

## 4. Pattern: Scheduled Notifications (Startup)

Use `trigger_startup` to initialize timers or periodic tasks. Note the startup handler runs at bot launch and has **no message context**.

### Startup-Compatible Nodes

| Node | Use in Startup? | Notes |
|:---|:---:|:---|
| `var_define` | ✅ Yes | Initialize global variables |
| `var_set` | ✅ Yes | Set default values |
| `sys_cmd` | ✅ Yes | Launch companion processes |
| `sys_http` | ✅ Yes | Fetch initial data from API |
| `logic_wait` | ✅ Yes | Delay startup sequence |
| `logic_if` | ✅ Yes | Conditional initialization |
| `action_msg` | ❌ No | No chat context at startup |
| `action_ban` | ❌ No | No message context |

### Example: Daily Greeting Timer

```
[Bot Startup] ──► [Define: greeting_time = "09:00"]
                   │
                   ▼
              [Define: is_initialized = True]
```

> **Note:** True scheduling (cron-like) requires external tools. The startup handler can initialize state that triggers actions when users interact.

---

## 5. Variable Interpolation Guide

### Syntax Reference

| Syntax | Scope | Example | Output |
|:---|:---|:---|:---|
| `{[var_name]}` | State variables | `Hello {[username]}` | `Hello John` |
| `{var_name}` | State variables | `Score: {score}` | `Score: 100` |

### Where Interpolation Works

| Field | Interpolation Support |
|:---|:---:|
| Message text (`action_msg`) | ✅ Full |
| Reply text (`action_reply`) | ✅ Full |
| Caption (`action_media`) | ✅ Full |
| Edit text (`action_edit`) | ✅ Full |
| Error message (`trigger_error_handler`) | ✅ Full |
| Condition expression (`logic_if`) | ❌ Raw Python |
| Command name (`trigger_cmd`) | ❌ No |

### Common Interpolation Patterns

```
# User greeting
Welcome, {[user_first_name]}!

# Dynamic content based on state
Your score is {[score]} out of {[max_score]}

# Conditional display (using python-telegram-bot features)
Status: {"✅ Active" if state.get('is_admin') else "❌ Inactive"}

# Nested state access
Last login: {[last_login_date]}
```

### Interpolation vs. Raw Python

- `{[var]}` / `{var}` are replaced with `state.get('var', '')` in an f-string
- For raw Python expressions in `logic_if`, use standard Python syntax directly (e.g., `state.get('count', 0) > 5`)

---

## 6. Error Handling Best Practices

### Always Use Error Handlers

Every trigger node should have an Error Handler connected. The auto-pairing feature does this automatically, but verify the connection:

```
✅ Correct:
[Trigger] ──┬──► [Main flow]
             └──► [Error Handler]

❌ Wrong:
[Trigger] ──► [Main flow] (no error handler)
```

### Error Message Templates

| Template | When to Use |
|:---|:---|
| `⚠️ Error: {error}` | General purpose |
| `Bot encountered an issue: {error}` | User-friendly |
| `[DEBUG] {error}` | Development/testing |
| `Admin notified. Error: {error}` | + send to admin chat |

### Defensive Programming

- Always check `message` exists before using it (the compiler does this)
- Use `logic_if` to validate state before accessing variables
- Set default values with `var_define` at startup or before use

---

## 7. Performance Optimization

### Bot Runtime

| Optimization | How |
|:---|:---|
| **Reduce wait nodes** | Each `logic_wait` blocks the entire handler for that user. Use only when necessary. |
| **Limit loop iterations** | Avoid high iteration counts in `logic_loop`. Consider using `range(100)` instead of `range(10000)`. |
| **Async HTTP** | `sys_http` is already async — multiple requests won't block each other across different chat sessions. |
| **Keyboard size** | Large keyboards (>10 buttons) increase message size. Break into multiple messages if needed. |

### Compiler Performance

The compiler itself is lightweight — graph traversal is O(n) where n is the number of connected nodes. For graphs with hundreds of nodes, compilation completes in milliseconds.

---

## 8. Production Deployment

### Option 1: Run from TG;BD

1. Build and test your bot in the Live Test modal
2. Click **Compile & Download** to save the `.py` file
3. Run the generated script directly:
   ```bash
   python tg_bot_build.py
   ```

### Option 2: Host on a VPS

1. Transfer the generated `.py` file to your server
2. Install dependencies:
   ```bash
   pip install python-telegram-bot aiohttp
   ```
3. Run with process manager:
   ```bash
   # Using systemd or supervisord
   [program:mybot]
   command=python /path/to/tg_bot_build.py
   user=youruser
   autostart=true
   autorestart=true
   ```

### Option 3: Docker Deployment

```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install python-telegram-bot aiohttp
COPY tg_bot_build.py .
CMD ["python", "tg_bot_build.py"]
```

```bash
docker build -t my-telegram-bot .
docker run -d --name mybot my-telegram-bot
```

### Environment Considerations

| Factor | Recommendation |
|:---|:---|
| **Python version** | 3.8 or higher |
| **Memory** | 128MB minimum for simple bots |
| **Polling** | Ensure outbound HTTPS to `api.telegram.org` |
| **File system** | Temp directory writable (for Live Test) |

---

<p align="center">
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/%C2%AB_Previous-API_%26_IPC-blue?style=flat-square" alt="Previous"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/Next-%C2%BB_Troubleshooting-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Active-blue?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
