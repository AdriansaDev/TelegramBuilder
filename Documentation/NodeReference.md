# 🧩 Node Reference

**31 node types** organized into 6 categories: Trigger, Action, Logic, Variable, System, and Error. Every node, its configuration fields, input/output ports, and generated Python code are documented below.

---

## 📖 Table of Contents

- [1. Trigger Nodes](#1-trigger-nodes)
  - [Bot Startup](#bot-startup)
  - [Command Trigger](#command-trigger)
  - [Any Message Trigger](#any-message-trigger)
  - [New Chat Member](#new-chat-member)
  - [Member Left](#member-left)
- [2. Action Nodes](#2-action-nodes)
  - [Send Message](#send-message)
  - [Reply to Message](#reply-to-message)
  - [Send Multimedia](#send-multimedia)
  - [Edit Message Text](#edit-message-text)
  - [Delete Message](#delete-message)
  - [Ban User](#ban-user)
  - [Unban User](#unban-user)
  - [Pin Message](#pin-message)
  - [Unpin Message](#unpin-message)
  - [Mute Member](#mute-member)
  - [Unmute Member](#unmute-member)
  - [Send Chat Action](#send-chat-action)
  - [Leave Chat](#leave-chat)
  - [Export Invite Link](#export-invite-link)
  - [Set Chat Title](#set-chat-title)
  - [Set Description](#set-description)
  - [Get Members Count](#get-members-count)
- [3. Logic Nodes](#3-logic-nodes)
  - [If Condition](#if-condition)
  - [Delay (Wait)](#delay-wait)
  - [Loop Action](#loop-action)
- [4. Variable Nodes](#4-variable-nodes)
  - [Define Variable](#define-variable)
  - [Update Variable](#update-variable)
  - [Check Variable](#check-variable)
- [5. System Nodes](#5-system-nodes)
  - [Subprocess Exec](#subprocess-exec)
  - [HTTP Request](#http-request)
- [6. Error Handler](#6-error-handler)
  - [Error Handler](#error-handler)
- [7. Node Category Summary](#7-node-category-summary)

---

## 1. Trigger Nodes

Trigger nodes initiate a bot workflow. Each trigger corresponds to a specific event type in Telegram. All trigger nodes have **0 inputs** and **1 output**.

### Bot Startup

| Property | Value |
|:---|:---|
| **ID** | `trigger_startup` |
| **Category** | `trigger` |
| **Inputs** | 0 |
| **Outputs** | 1 (`output_1`) |
| **Config** | None |

Fires when the bot starts hosting, before any messages are processed. Connected to `application.post_init` in the generated code.

**Use cases:** Initialize variables, send startup notifications, load persistent data.

```python
# Generated handler:
async def startup_handler_123(application):
    try:
        # Nodes connected here are compiled via traverseStartup()
        # Only supports: var_define, var_set, sys_cmd, sys_http, logic_wait, logic_if
    except Exception as e:
        logging.error(f"Startup error: {e}")
```

> ⚠ **Startup node restriction:** Action nodes (send message, reply, etc.) are NOT supported inside startup flow because there is no chat context yet.

---

### Command Trigger

| Property | Value |
|:---|:---|
| **ID** | `trigger_cmd` |
| **Category** | `trigger` |
| **Inputs** | 0 |
| **Outputs** | 1 (`output_1`) |
| **Config** | `cmd` (text) |

Fires when a user sends a command like `/start` or `/help`.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `cmd` | Text | `start` | Command name (without `/`) |

```python
# Generated handler:
application.add_handler(CommandHandler("start", handler_123))
```

---

### Any Message Trigger

| Property | Value |
|:---|:---|
| **ID** | `trigger_msg` |
| **Category** | `trigger` |
| **Inputs** | 0 |
| **Outputs** | 1 (`output_1`) |
| **Config** | Content type checkboxes |

Fires when a user sends any message matching the selected content types. If no types are checked, defaults to `filters.TEXT`.

| Config Field | Type | Options |
|:---|:---|:---|
| `content_text` | Checkbox | 0/1 |
| `content_photo` | Checkbox | 0/1 |
| `content_video` | Checkbox | 0/1 |
| `content_document` | Checkbox | 0/1 |
| `content_audio` | Checkbox | 0/1 |
| `content_animation` | Checkbox | 0/1 (GIF) |
| `content_voice` | Checkbox | 0/1 |
| `content_sticker` | Checkbox | 0/1 |

```python
# Generated handler:
application.add_handler(MessageHandler(filters.TEXT | filters.PHOTO, handler_123))
```

---

### New Chat Member

| Property | Value |
|:---|:---|
| **ID** | `trigger_new_member` |
| **Category** | `trigger` |
| **Inputs** | 0 |
| **Outputs** | 1 (`output_1`) |
| **Config** | None |

Fires when a new user joins the chat.

```python
application.add_handler(MessageHandler(filters.StatusUpdate.NEW_CHAT_MEMBERS, handler_123))
```

---

### Member Left

| Property | Value |
|:---|:---|
| **ID** | `trigger_left_member` |
| **Category** | `trigger` |
| **Inputs** | 0 |
| **Outputs** | 1 (`output_1`) |
| **Config** | None |

Fires when a user leaves the chat.

```python
application.add_handler(MessageHandler(filters.StatusUpdate.LEFT_CHAT_MEMBER, handler_123))
```

---

## 2. Action Nodes

Action nodes perform Telegram operations. All action nodes have **1 input** and **1 output**, except those with inline keyboards which add dynamic button outputs.

### Send Message

| Property | Value |
|:---|:---|
| **ID** | `action_msg` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 + per-button outputs |
| **Keyboard** | ✅ Yes |

Sends a text message to the chat. The last sent message ID and chat ID are stored in `state['__last_bot_message_id']` and `state['__last_bot_chat_id']`.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `text` | Textarea | — | Message text. Supports `{[var]}` and `{var}` interpolation |
| `parse_mode` | Select | `Markdown` | `Markdown` or `HTML` |
| `keyboard_data` | JSON (auto) | — | Inline keyboard rows (built via visual editor) |

```python
# Generated code:
eval_text = f"""Hello, {state.get('username', '')}!"""
sent_msg = await context.bot.send_message(
    chat_id=message.chat.id, text=eval_text,
    parse_mode='Markdown', reply_markup=markup_123
)
state['__last_bot_message_id'] = sent_msg.message_id
state['__last_bot_chat_id'] = sent_msg.chat.id
```

---

### Reply to Message

| Property | Value |
|:---|:---|
| **ID** | `action_reply` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 + per-button outputs |
| **Keyboard** | ✅ Yes |

Replies directly to the triggering user's message.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `text` | Textarea | — | Reply text. Supports interpolation |
| `keyboard_data` | JSON (auto) | — | Inline keyboard |

```python
# Generated code:
sent_msg = await message.reply_text(
    eval_text, reply_to_message_id=message.message_id,
    reply_markup=markup_123
)
```

---

### Send Multimedia

| Property | Value |
|:---|:---|
| **ID** | `action_media` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 + per-button outputs |
| **Keyboard** | ✅ Yes |

Sends a photo, video, document, audio, or animation file.

| Config Field | Type | Default | Options |
|:---|:---|:---|:---|
| `media_type` | Select | `photo` | photo, video, document, audio, animation |
| `media_url` | Text | — | URL or file_id of the media |
| `caption` | Text | — | Caption text |
| `keyboard_data` | JSON (auto) | — | Inline keyboard |

```python
# Generated code:
await getattr(message, 'reply_photo')(
    "https://example.com/image.jpg",
    caption=f"Check this out!",
    reply_markup=markup_123
)
```

---

### Edit Message Text

| Property | Value |
|:---|:---|
| **ID** | `action_edit` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 + per-button outputs |
| **Keyboard** | ✅ Yes |

Edits an existing bot message. If `chat_id` and `msg_id` are left blank, uses the last bot message IDs stored in state.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `text` | Textarea | — | New text. Supports interpolation |
| `chat_id` | Text | *(auto)* | Target chat ID. Blank = last bot chat |
| `msg_id` | Text | *(auto)* | Target message ID. Blank = last bot message |
| `keyboard_data` | JSON (auto) | — | Inline keyboard |

```python
# Generated code:
edit_chat_id = state.get('__last_bot_chat_id', message.chat.id)
edit_message_id = state.get('__last_bot_message_id', message.message_id)
await context.bot.edit_message_text(
    eval_text, chat_id=edit_chat_id,
    message_id=edit_message_id, reply_markup=markup_123
)
```

---

### Delete Message

| Property | Value |
|:---|:---|
| **ID** | `action_delete` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Deletes a message.

| Config Field | Type | Default | Options |
|:---|:---|:---|:---|
| `target` | Select | `trigger` | `trigger` (user's message), `bot_last` (last bot msg), `specific_id` |
| `msg_id` | Text | — | Message ID (only for `specific_id`) |

```python
# target = 'trigger':
await message.delete()

# target = 'bot_last':
pass  # Bot messages can't always be deleted; handled at runtime

# target = 'specific_id':
await context.bot.delete_message(
    chat_id=message.chat.id, message_id=int(12345)
)
```

---

### Ban User

| Property | Value |
|:---|:---|
| **ID** | `action_ban` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Bans a user from the chat.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `target` | Select | `trigger` | `trigger` (user who sent message) or `specific_id` |
| `user_id` | Text | — | User ID (only for `specific_id`) |

```python
await context.bot.ban_chat_member(
    chat_id=message.chat.id, user_id=int(message.from_user.id)
)
```

---

### Unban User

| Property | Value |
|:---|:---|
| **ID** | `action_unban` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Unbans a previously banned user.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `target` | Select | `trigger` | `trigger` or `specific_id` |
| `user_id` | Text | — | User ID (only for `specific_id`) |

```python
await context.bot.unban_chat_member(
    chat_id=message.chat.id, user_id=int(message.from_user.id)
)
```

---

### Pin Message

| Property | Value |
|:---|:---|
| **ID** | `action_pin` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Pins a message in the chat.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `target` | Select | `trigger` | `trigger` (user's message) or `specific_id` |
| `msg_id` | Text | — | Message ID (only for `specific_id`) |

```python
await context.bot.pin_chat_message(
    chat_id=message.chat.id, message_id=int(message.message_id)
)
```

---

### Unpin Message

| Property | Value |
|:---|:---|
| **ID** | `action_unpin` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Unpins a specific message or all messages.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `target` | Select | `trigger` | `trigger`, `specific_id`, or `all` |
| `msg_id` | Text | — | Message ID (only for `specific_id`) |

```python
# target = 'all':
await context.bot.unpin_all_chat_messages(chat_id=message.chat.id)

# target = 'trigger':
await context.bot.unpin_chat_message(
    chat_id=message.chat.id, message_id=int(message.message_id)
)
```

---

### Mute Member

| Property | Value |
|:---|:---|
| **ID** | `action_mute` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Restricts a member from sending messages.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `user_id` | Text | *(trigger user)* | User ID to mute |
| `minutes` | Number | `0` | Duration in minutes. `0` = permanent |

```python
# With duration:
until = datetime.datetime.now() + datetime.timedelta(minutes=30)
await context.bot.restrict_chat_member(
    chat_id=message.chat.id, user_id=int(targetUid),
    permissions=ChatPermissions(can_send_messages=False),
    until_date=until
)

# Permanent (minutes = 0):
await context.bot.restrict_chat_member(
    chat_id=message.chat.id, user_id=int(targetUid),
    permissions=ChatPermissions(can_send_messages=False)
)
```

---

### Unmute Member

| Property | Value |
|:---|:---|
| **ID** | `action_unmute` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Restores full permissions to a previously muted member.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `user_id` | Text | *(trigger user)* | User ID to unmute |

```python
await context.bot.restrict_chat_member(
    chat_id=message.chat.id, user_id=int(targetUid),
    permissions=ChatPermissions(
        can_send_messages=True, can_send_media_messages=True,
        can_send_polls=True, can_send_other_messages=True,
        can_add_web_page_previews=True, can_change_info=True,
        can_invite_users=True, can_pin_messages=True
    )
)
```

---

### Send Chat Action

| Property | Value |
|:---|:---|
| **ID** | `action_chataction` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Sends a chat action indicator (e.g., "typing...").

| Config Field | Type | Default | Options |
|:---|:---|:---|:---|
| `action_type` | Select | `typing` | typing, upload_photo, record_video |

```python
await context.bot.send_chat_action(
    chat_id=message.chat.id, action="typing"
)
```

---

### Leave Chat

| Property | Value |
|:---|:---|
| **ID** | `action_leave` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Bot leaves the chat.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `chat_id` | Text | *(current chat)* | Chat ID to leave |

```python
await context.bot.leave_chat(chat_id=message.chat.id)
```

---

### Export Invite Link

| Property | Value |
|:---|:---|
| **ID** | `action_export_link` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Exports a chat invite link and saves it to a state variable.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `save_var` | Text | `invite_url` | State variable name to store the link |

```python
state["invite_url"] = await context.bot.export_chat_invite_link(
    chat_id=message.chat.id
)
```

---

### Set Chat Title

| Property | Value |
|:---|:---|
| **ID** | `action_set_title` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Changes the chat title.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `title` | Text | `Group` | New chat title |

```python
await context.bot.set_chat_title(
    chat_id=message.chat.id, title="My Super Group"
)
```

---

### Set Description

| Property | Value |
|:---|:---|
| **ID** | `action_set_desc` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Changes the chat description.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `desc` | Textarea | — | New chat description |

```python
await context.bot.set_chat_description(
    chat_id=message.chat.id,
    description="""Welcome to our group!"""
)
```

---

### Get Members Count

| Property | Value |
|:---|:---|
| **ID** | `action_get_members` |
| **Category** | `action` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Gets the number of members in the chat and saves to a variable.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `save_var` | Text | `count` | State variable name to store the count |

```python
state["count"] = await context.bot.get_chat_member_count(
    chat_id=message.chat.id
)
```

---

## 3. Logic Nodes

Logic nodes control the flow of execution. They introduce branching, waiting, and looping.

### If Condition

| Property | Value |
|:---|:---|
| **ID** | `logic_if` |
| **Category** | `logic` |
| **Inputs** | 1 |
| **Outputs** | 2 (`output_1` = True, `output_2` = False) |

Evaluates a Python expression at runtime.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `condition` | Text | `True` | Any valid Python expression |

**Output wiring:**
```
        ┌─────────┐
   ────▶│   If    │
        │  cond   │
        └────┬────┘
     True    │    False
  output_1   │   output_2
        ┌────▼─┐  ┌────▼──┐
        │ True  │  │ False │
        │ Path  │  │ Path  │
        └───────┘  └───────┘
```

```python
# Generated code:
if message.text == 'ping':
    # True branch nodes
else:
    # False branch nodes
```

**Common conditions:**
| Expression | Behavior |
|:---|:---|
| `message.text == 'hello'` | True if message text is exactly "hello" |
| `state.get('count', 0) > 5` | True if count variable exceeds 5 |
| `message.chat.type == 'supergroup'` | True only in supergroups |
| `'keyword' in message.text` | True if message contains "keyword" |

---

### Delay (Wait)

| Property | Value |
|:---|:---|
| **ID** | `logic_wait` |
| **Category** | `logic` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Pauses execution for a specified duration.

| Config Field | Type | Default | Options |
|:---|:---|:---|:---|
| `duration` | Number | `1` | Duration value |
| `unit` | Select | `seconds` | milliseconds, seconds, minutes, hours |

```python
# 5 seconds:
await asyncio.sleep(float(5) * 1)

# 500 milliseconds:
await asyncio.sleep(float(500) * 0.001)

# 2 minutes:
await asyncio.sleep(float(2) * 60)
```

---

### Loop Action

| Property | Value |
|:---|:---|
| **ID** | `logic_loop` |
| **Category** | `logic` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Loops through the connected node N times with a per-iteration delay. After the loop completes, execution continues to the next node in the main chain.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `iterations` | Text | `5` | Number of loop iterations (Python expression) |
| `loop_delay` | Number | `1.0` | Delay between iterations (seconds) |

```
        ┌──────────┐
   ────▶│   Loop   │
        │ 5 times  │
        └────┬─────┘
             │ output_1
             ▼
        ┌──────────┐
   ────▶│ Send Msg │  ← Runs 5 times with 1s delay
        └──────────┘
             │
             ▼
        ┌──────────┐
        │ Continue  │  ← After loop completes
        │  Chain    │
        └──────────┘
```

```python
# Generated code:
for i in range(int(5)):
    # Loop body (connected node)
    await asyncio.sleep(float(1.0))
# Continue main chain after loop
```

---

## 4. Variable Nodes

Variable nodes manage session state. Variables are stored per-chat in `chat_states[chat_id]`.

### Define Variable

| Property | Value |
|:---|:---|
| **ID** | `var_define` |
| **Category** | `var` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Declares a variable with a specific type and initial value.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `var_key` | Text | — | Variable name (used as state key) |
| `var_type` | Select | `Number` | Number, Float, String, Boolean, Character |
| `var_val` | Text | `0` | Initial value (format depends on type) |

```python
# Generated code:
state["username"] = "JohnDoe"     # String
state["is_admin"] = True           # Boolean
state["score"] = int(100)          # Number
state["ratio"] = float(3.14)       # Float
```

---

### Update Variable

| Property | Value |
|:---|:---|
| **ID** | `var_set` |
| **Category** | `var` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Updates an existing variable. The value is formatted according to the selected type.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `var_key` | Text | — | Variable name |
| `var_type` | Select | `String` | Number, Float, String, Boolean, Character |
| `var_val` | Text | `""` | New value (Python expression or literal) |

```python
# Generated code:
state["score"] = int(50)           # Number
state["name"] = "Alice"            # String
state["active"] = False            # Boolean
state["price"] = float(19.99)      # Float
```

> **Type formatting:** `var_set` uses the same `formatValue()` logic as `var_define`. Strings are quoted, booleans are capitalized (`True`/`False`), numbers get `int()`/`float()` casts.

---

### Check Variable

| Property | Value |
|:---|:---|
| **ID** | `var_get` |
| **Category** | `var` |
| **Inputs** | 1 |
| **Outputs** | 2 (`output_1` = Yes/Equal, `output_2` = No/Not Equal) |

Checks if a variable equals a specified value and branches accordingly.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `var_key` | Text | — | Variable name to check |
| `var_val` | Text | `""` | Value to compare against |

```
        ┌──────────┐
   ────▶│ Check    │
        │ Variable │
        └─────┬────┘
     Yes     │     No
  output_1   │   output_2
        ┌────▼─┐  ┌────▼──┐
        │ Match │  │ No    │
        │ Path  │  │ Match │
        └───────┘  └───────┘
```

```python
# Generated code:
if state.get("score") == 100:
    # Yes path
else:
    # No path
```

---

## 5. System Nodes

System nodes interact with the operating system and external services.

### Subprocess Exec

| Property | Value |
|:---|:---|
| **ID** | `sys_cmd` |
| **Category** | `sys` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Executes a system command via `subprocess.Popen`. The command is sanitized with `shlex.split()` to prevent shell injection.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `cmd` | Text | — | Command to execute (e.g., `echo Hello`, `notepad.exe`) |

```python
# Generated code:
cmd_parts = shlex.split("echo Hello World")
subprocess.Popen(cmd_parts)
```

> ⚠ **Security:** The command is split via `shlex` to prevent argument injection. Shell operators (`|`, `>`, `&&`) are treated as literal arguments, not shell features. For complex shell pipelines, wrap the command in your shell's explicit path (e.g., `cmd.exe /c "dir | findstr .py"` on Windows).

---

### HTTP Request

| Property | Value |
|:---|:---|
| **ID** | `sys_http` |
| **Category** | `sys` |
| **Inputs** | 1 |
| **Outputs** | 1 |

Makes an HTTP request and saves the JSON response to a state variable.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `method` | Select | `GET` | GET or POST |
| `url` | Text | — | Request URL |
| `save_var` | Text | `res` | State variable name for the response |

```python
# Generated code:
async with aiohttp.ClientSession() as session:
    async with session.get("https://api.example.com/data") as response:
        state["res"] = await response.json()
```

> **Note:** The default `save_var` is `res`. After the request completes, you can reference `{[res]}` in message text to include the response data.

---

## 6. Error Handler

### Error Handler

| Property | Value |
|:---|:---|
| **ID** | `trigger_error_handler` |
| **Category** | `error` |
| **Inputs** | 1 |
| **Outputs** | 0 (terminus) |

Catches exceptions from its paired trigger node and sends a formatted error message. Automatically created when any trigger node is spawned.

| Config Field | Type | Default | Description |
|:---|:---|:---|:---|
| `error_msg` | Textarea | — | Error message template. Use `{error}` for the exception text |
| `error_chat_id` | Text | *(current chat)* | Target chat for the error notification |

```python
# Generated code (inside trigger handler):
except Exception as e:
    try:
        error_text = f"Something went wrong: {e}"
        if message:
            await context.bot.send_message(
                chat_id=message.chat.id, text=error_text
            )
    except Exception:
        logging.error(f"Error in handler_123: {e}")
```

**Error message templates:**
| Template | Rendered Output |
|:---|:---|
| `⚠️ Error: {error}` | ⚠️ Error: 'NoneType' object has no attribute 'text' |
| `Bot crashed! Details: {error}` | Bot crashed! Details: ... |

---

## 7. Node Category Summary

| Category | Color | Nodes | Purpose |
|:---|:---:|:---:|:---|
| **Trigger** | 🔴 Red | 5 | Event listeners (commands, messages, joins, leaves, startup) |
| **Action** | 🔵 Blue | 16 | Telegram API operations |
| **Logic** | 🟢 Green | 3 | Flow control (conditions, waits, loops) |
| **Variable** | 🟠 Orange | 3 | State management (define, update, check) |
| **System** | 🟣 Purple | 2 | OS commands, HTTP requests |
| **Error** | ⚪ Gray | 1 | Exception handling |

---

<p align="center">
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/%C2%AB_Previous-First_Moves-blue?style=flat-square" alt="Previous"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/Next-%C2%BB_Compiler_Architecture-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Gray-darkgrey?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Active-blue?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
