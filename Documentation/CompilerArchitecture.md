# ⚙️ Compiler Architecture

The **TG;BD Compiler** (`compiler.js`) transforms a visual node graph into a fully functional Python Telegram bot. This document explains the compilation pipeline, code generation strategy, and architecture design.

---

## 📖 Table of Contents

- [1. Overview](#1-overview)
- [2. Compilation Pipeline](#2-compilation-pipeline)
- [3. The Compiler Class](#3-the-compiler-class)
- [4. Traversal Strategies](#4-traversal-strategies)
- [5. Code Generation per Node Type](#5-code-generation-per-node-type)
- [6. Session State Management](#6-session-state-management)
- [7. Helper Methods](#7-helper-methods)
- [8. Generated Output Structure](#8-generated-output-structure)

---

## 1. Overview

```
┌───────────────────────────────────────────────────────┐
│                   compiler.js                         │
│                                                       │
│  Compiler(graph, token)                               │
│    ├── compile()      → generates full Python script   │
│    ├── traverse()     → recursive node walker         │
│    ├── traverseStartup() → startup-specific walker    │
│    └── *_helpers()    → interpolation, formatting      │
│                                                       │
│  Input:                                               │
│    graph = editor.drawflow.Home.data (node map)       │
│    token = Telegram Bot API token                     │
│                                                       │
│  Output:                                              │
│    Single .py file with imports, handlers, main()     │
└───────────────────────────────────────────────────────┘
```

### Design Principles

| Principle | Implementation |
|:---|:---|
| **Stateless Compilation** | The compiler takes a graph snapshot and produces code — no side effects |
| **Session Isolation** | Per-chat state via `chat_states[chat_id]`, not global variables |
| **Safe Code Generation** | All user values are properly quoted/escaped/formatted |
| **Full Output** | No code truncation; complete functions are always emitted |
| **No Dead Code** | Only generates handlers for connected ports |

---

## 2. Compilation Pipeline

The compilation process follows a strict sequence:

### Phase 1: Preamble (lines 9–33)

```
compile() start
    │
    ├── Push standard Python imports
    ├── Push logging config
    ├── Push TOKEN constant
    ├── Push chat_states dict
    └── Push SafeMessage helper class
```

### Phase 2: Handler Generation (lines 37–86)

For each node with `name.startsWith('trigger_')`:

```
    │
    ├── Is trigger_startup?
    │   └── Generate async def startup_handler_{id}(application):
    │       └── try: traverseStartup() → except: logging
    │
    ├── Is trigger_cmd?
    │   └── Generate async def handler_{id}(update, context):
    │       └── try: setup state, SafeMessage → traverse()
    │           → except: error handler or logging
    │
    ├── Is trigger_msg / trigger_new_member / trigger_left_member?
    │   └── (same pattern as trigger_cmd)
    │
    └── (trigger_error_handler is skipped — handled inline)
```

### Phase 3: Button Handler Generation (lines 88–115)

For each message-type node with `keyboard_data`:

```
    │
    └── Parse keyboard_data JSON
        └── For each button with a connected output port:
            └── Generate async def handler_btn_{id}_{idx}(update, context):
                └── try: setup → traverse() → except: error handling
```

### Phase 4: Main Function (lines 117–176)

```
    │
    ├── application = Application.builder().token(TOKEN).build()
    ├── If trigger_startup exists → application.post_init = handler
    ├── Register all CommandHandler, MessageHandler
    ├── Register all CallbackQueryHandler for buttons
    ├── print("Engine Subprocess Injector Active...")
    ├── application.run_polling()
    └── if __name__ == '__main__': main()
```

### Phase 5: Assembly (line 177)

```javascript
return this.code.join("\n");
```

All code lines are joined with newlines into a complete Python script.

---

## 3. The Compiler Class

```javascript
class Compiler {
    constructor(graph, token) { ... }
    compile() { ... }
    traverse(nodeId, indentLvl) { ... }
    traverseStartup(nodeId, indentLvl) { ... }
    getFirstNonErrorConnection(trigger) { ... }
    findErrorHandlerForTrigger(trigger) { ... }
    interpolate(text) { ... }
    formatValue(valExpr, vtype) { ... }
    genKeyboard(nodeId, indent) { ... }
}
```

### Class Properties

| Property | Type | Description |
|:---|:---|:---|
| `this.graph` | Object | Drawflow node map (`{ [id]: nodeData }`) |
| `this.token` | String | Telegram bot token |
| `this.code` | String[] | Output buffer (joined at end of `compile()`) |

### Constructor

```javascript
constructor(graph, token) {
    this.graph = graph.drawflow.Home.data;
    this.token = token || 'YOUR_TOKEN_HERE';
    this.code = [];
}
```

The graph is extracted from the Drawflow export structure. If no token is provided, a placeholder is used (user is prompted to set it).

---

## 4. Traversal Strategies

### 4.1 `traverse(nodeId, indentLvl)` — Main Flow Walker

This method walks the graph recursively, emitting Python code at each node.

```
traverse(nodeId, indentLvl)
    │
    ├── Look up node in graph
    ├── If node not found → push pass and return  ← Safety valve
    ├── Determine indent string ("    ".repeat(indentLvl))
    │
    ├── Switch on node.name:
    │   ├── action_msg / action_reply / action_media
    │   ├── action_edit / action_delete / action_ban
    │   ├── action_unban / action_pin / action_unpin
    │   ├── action_mute / action_unmute / action_leave
    │   ├── action_chataction / action_export_link
    │   ├── action_set_title / action_set_desc / action_get_members
    │   ├── logic_wait / logic_loop / logic_if
    │   ├── var_define / var_set / var_get
    │   ├── sys_cmd / sys_http
    │   └── (unknown types are silently skipped)
    │
    ├── If node is NOT a branching type (not logic_if/var_get/logic_loop):
    │   └── Follow output_1 to next node → recursive traverse()
    │
    └── Return
```

### 4.2 `traverseStartup(nodeId, indentLvl)` — Startup Walker

A restricted version of `traverse` used exclusively inside `trigger_startup` handlers. It only supports:

- `var_define` / `var_set` (stores to `chat_states.setdefault('global', {})`)
- `sys_cmd` (same as main traverse)
- `sys_http` (stores to global state)
- `logic_wait` (same as main traverse)
- `logic_if` (recursive traversal via `traverseStartup`)

> **Why restricted?** Action nodes like `Send Message` require a `message` context which doesn't exist during startup. Only storage and system operations make sense.

### 4.3 Branching Special Cases

| Node Type | Traversal Behavior |
|:---|:---|
| `logic_if` | Traverses both True (`output_1`) and False (`output_2`) paths. Calls `return` to prevent fall-through. |
| `var_get` | Traverses both Yes (`output_1`) and No (`output_2`) paths. Calls `return`. |
| `logic_loop` | Traverses loop body first, adds sleep, then traverses the **same connection** again for post-loop continuation. Calls `return`. |

---

## 5. Code Generation per Node Type

Each node type produces specific Python code. Here is the complete mapping:

| Node ID | Generated Code Pattern |
|:---|:---|
| `action_msg` | `await context.bot.send_message(chat_id=..., text=f"...", parse_mode='...', reply_markup=...)` |
| `action_reply` | `await message.reply_text(f"...", reply_to_message_id=..., reply_markup=...)` |
| `action_media` | `await getattr(message, 'reply_{type}')(url, caption=f"...", reply_markup=...)` |
| `action_edit` | `await context.bot.edit_message_text(f"...", chat_id=..., message_id=..., reply_markup=...)` |
| `action_delete` | `await message.delete()` / `pass` / `await context.bot.delete_message(...)` |
| `action_ban` | `await context.bot.ban_chat_member(chat_id=..., user_id=int(...))` |
| `action_unban` | `await context.bot.unban_chat_member(chat_id=..., user_id=int(...))` |
| `action_pin` | `await context.bot.pin_chat_message(chat_id=..., message_id=int(...))` |
| `action_unpin` | `await context.bot.unpin_all_chat_messages(...)` / `unpin_chat_message(...)` |
| `action_mute` | `await context.bot.restrict_chat_member(..., permissions=ChatPermissions(can_send_messages=False), until_date=...)` |
| `action_unmute` | `await context.bot.restrict_chat_member(..., permissions=ChatPermissions(can_send_messages=True, ...))` |
| `action_chataction` | `await context.bot.send_chat_action(chat_id=..., action="...")` |
| `action_leave` | `await context.bot.leave_chat(chat_id=...)` |
| `action_export_link` | `state["var"] = await context.bot.export_chat_invite_link(chat_id=...)` |
| `action_set_title` | `await context.bot.set_chat_title(chat_id=..., title="...")` |
| `action_set_desc` | `await context.bot.set_chat_description(chat_id=..., description="""...""")` |
| `action_get_members` | `state["var"] = await context.bot.get_chat_member_count(chat_id=...)` |
| `logic_wait` | `await asyncio.sleep(float(val) * multiplier)` |
| `logic_loop` | `for i in range(int(N)):` → body → `await asyncio.sleep(float(delay))` → post-loop continuation |
| `logic_if` | `if condition:` → True branch → `else:` → False branch |
| `var_define` | `state["key"] = formatted_value` |
| `var_set` | `state["key"] = formatted_value` |
| `var_get` | `if state.get("key") == val:` → Yes branch → `else:` → No branch |
| `sys_cmd` | `cmd_parts = shlex.split("...")` → `subprocess.Popen(cmd_parts)` |
| `sys_http` | `async with aiohttp.ClientSession() as session:` → `async with session.get("url") as response:` → `state["var"] = await response.json()` |

---

## 6. Session State Management

### Per-Chat Isolation

```python
chat_states = {}
```

Every handler initializes state from this global dictionary:

```python
state = chat_states.setdefault(chat_id, {}) if chat_id else {}
```

This ensures:
- **User A** and **User B** have independent state
- State persists across messages within the same session
- No cross-chat data leaks

### Startup Global State

For `trigger_startup` handlers, state is stored in a global key:

```python
chat_states.setdefault('global', {})["var_name"] = value
```

### State Tracking

The system automatically tracks the last bot message for `Edit Message Text`:

```python
state['__last_bot_message_id'] = sent_msg.message_id
state['__last_bot_chat_id'] = sent_msg.chat.id
```

> **Reserved state keys:** `__last_bot_message_id` and `__last_bot_chat_id` are used internally. Avoid using these as variable names.

---

## 7. Helper Methods

### `getFirstNonErrorConnection(trigger)`

Returns the first `output_1` connection that is NOT an error handler. This allows the error handler to be wired alongside the main flow.

```javascript
getFirstNonErrorConnection(trigger) {
    const output = trigger.outputs.output_1;
    if (!output || !output.connections) return null;
    for (const conn of output.connections) {
        const targetNode = this.graph[conn.node];
        if (targetNode && targetNode.name !== 'trigger_error_handler') {
            return conn;
        }
    }
    return null;
}
```

### `findErrorHandlerForTrigger(trigger)`

Returns the error handler node connected to `output_1`, or `null` if none exists.

### `interpolate(text)`

Converts template variables into Python f-string expressions:

| Template Syntax | Python Output |
|:---|:---|
| `Hello {[name]}` | `f"Hello {state.get('name', '')}"` |
| `Score: {score}` | `f"Score: {state.get('score', '')}"` |
| Raw `{` without var | Left unchanged |

Supports both `{[var]}` (explicit bracket syntax) and `{var}` (simple syntax) to avoid conflicts with Python f-string escaping.

### `formatValue(valExpr, vtype)`

Formats a value according to its declared type:

| Type | Input | Output |
|:---|:---|:---|
| `String` | `Hello` | `"Hello"` |
| `Boolean` | `true` | `True` |
| `Boolean` | `false` | `False` |
| `Float` | `3.14` | `float(3.14)` |
| `Number` | `42` | `int(42)` |
| *(other)* | `expression` | `expression` (raw) |

### `genKeyboard(nodeId, indent)`

Parses `keyboard_data` JSON and generates an `InlineKeyboardMarkup`:

```python
markup_123 = InlineKeyboardMarkup([
    [InlineKeyboardButton("Yes", callback_data="btn_yes"),
     InlineKeyboardButton("No", callback_data="btn_no")],
    [InlineKeyboardButton("Maybe", callback_data="btn_maybe")],
])
```

Returns the markup variable name for use in message/reply calls, or `'None'` if no keyboard data exists.

---

## 8. Generated Output Structure

A complete generated script has this anatomy:

```python
# === IMPORTS ===
import asyncio
import logging
import shlex
import subprocess
import datetime
import aiohttp
from telegram import ..., ChatPermissions
from telegram.ext import ...

# === CONFIG ===
logging.basicConfig(...)
TOKEN = "1234567890:ABC..."
chat_states = {}

# === HELPERS ===
class SafeMessage:
    # Wraps message + user for safe attribute access
    ...

# === TRIGGER HANDLERS ===
async def handler_123(update, context):
    try:
        chat_id = ...
        state = chat_states.setdefault(chat_id, {})
        message = SafeMessage(update.message, update.effective_user)
        # Compiled node chain...
    except Exception as e:
        # Error handler or logging...

# === STARTUP HANDLER ===
async def startup_handler_456(application):
    try:
        # Compiled startup chain...
    except Exception as e:
        logging.error(f"Startup error: {e}")

# === BUTTON HANDLERS ===
async def handler_btn_789_1(update, context):
    try:
        query = update.callback_query
        await query.answer()
        # Compiled button handler chain...
    except Exception as e:
        # Error handling...

# === MAIN ===
def main():
    application = Application.builder().token(TOKEN).build()
    # application.post_init = startup_handler...
    # application.add_handler(CommandHandler(...))
    # application.add_handler(MessageHandler(...))
    # application.add_handler(CallbackQueryHandler(...))
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == '__main__':
    main()
```

---

## 🔧 Extending the Compiler

To add a new node type, follow these steps:

1. **Register the node** in `nodes.js` (`NODE_REGISTRY`)
2. **Add the handler** in `compiler.js` inside `traverse()`:
   ```javascript
   else if (node.name === 'your_new_node') {
       this.code.push(`${indent}# Your generated Python code`);
   }
   ```
3. **Handle startup** path in `traverseStartup()` if applicable
4. **Register any handlers** in `compile()` (main function section)

---

<p align="center">
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/%C2%AB_Previous-Node_Reference-blue?style=flat-square" alt="Previous"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/Next-%C2%BB_API_%26_IPC-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Active-blue?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
