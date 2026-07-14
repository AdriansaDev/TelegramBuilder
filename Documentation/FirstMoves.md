# 🖱️ First Moves & Core Architecture

**Telegram Builder (TG;BD)** uses a dual-input paradigm for rapid canvas navigation, viewport transformations, and node-graph orchestration. The design decouples visual state rendering from the underlying telemetry compilation.

---

## 📖 Table of Contents

- [1. Input Mapping Specification](#1-input-mapping-specification)
- [2. Workspace Initialization & Contextual Workflows](#2-workspace-initialization--contextual-workflows)
- [3. Viewport Navigation & Scale Control](#3-viewport-navigation--scale-control)
- [4. Keyboard Combinations](#4-keyboard-combinations)
- [5. UI Component Reference](#5-ui-component-reference)
- [6. Node Wiring & Connections](#6-node-wiring--connections)
- [7. Inline Keyboard Builder](#7-inline-keyboard-builder)
- [8. Zoom & Viewport Controls](#8-zoom--viewport-controls)

---

## 1. Input Mapping Specification

| Input Action | Target Component | Functional Outcome | Context / Constraints |
| :--- | :--- | :--- | :--- |
| **Right-Click** (`Mouse_Button_2`) | Canvas workspace | Instantiates the contextual node creation menu | Inhibited when hovering active node hitboxes |
| **Left-Click** (`Mouse_Button_1`) | UI elements / nodes | Executes widget primitives, manages selection states, handles node-to-node edge mapping | Supports multi-selection via bounding box marquee dragging |
| **Middle-Click Drag** (`Mouse_Button_3`) | Canvas workspace | Translates the viewport camera matrix (panning) | Active globally across all canvas coordinates |
| **Scroll Wheel** (`Mouse_Wheel`) | Canvas workspace | Modulates viewport scale factor (zooming) | Anchored to the current screen-space cursor coordinate |
| **Left-Click Drag** (from output circle) | Node output → canvas | Creates a new connection wire | Release over another node's input to complete |

---

## 2. Workspace Initialization & Contextual Workflows

### 2.1 Node Instantiation Workflow

Triggering `Mouse_Button_2` over any vacant coordinate within the viewport dispatches an event to the UI manager, dynamically spawning the context-sensitive node creation menu. This interface acts as the primary gateway to the node registry.

```
Right-click on canvas
       │
       ▼
┌─────────────────────────────┐
│  Node Creation Menu         │
│  ─────────────────────      │
│  🔍 [Search nodes...]       │
│                             │
│  ⚡ Trigger                 │
│    ● Bot Startup            │
│    ● Command Trigger        │
│    ● Any Message Trigger    │
│    ● New Chat Member        │
│    ● Member Left            │
│                             │
│  🎯 Action                  │
│    ● Send Message           │
│    ● Reply to Message       │
│    ● Send Multimedia        │
│    ● ...                    │
└─────────────────────────────┘
       │
       ▼
Node placed at cursor position
```

- **Coordinate Resolution:** Instantiation coordinates map directly from viewport space to canvas world space, respecting the current zoom level.
- **State Management:** Opening the menu flags the canvas state as `Interaction_State: Interrupted` to prevent accidental panning during selection.
- **Search Filter:** Typing in the search bar filters visible node names in real-time. The menu closes on `Escape` or clicking outside.

### 2.2 Selection & Component Manipulation

Executing `Mouse_Button_1` triggers distinct interaction pipelines depending on cursor intersection with workspace entities.

- **Single Entity Selection:** A discrete click on an active node hitbox clears the previous selection registry and pushes the target entity's unique identifier to the focus stack, transitioning its visual state to `State: Selected` (highlighted border).
- **Marquee Selection (Bounding Box Drag):** Dragging the cursor while maintaining `Mouse_Button_1` over an empty canvas coordinate instantiates a dynamic 2D marquee rectangle. All nodes whose position vectors intersect the boundary are appended to the active selection array.
- **Node Translation (Dragging):** Maintaining `Mouse_Button_1` over a selected node initializes coordinate synchronization. The entity's translation vectors are recalculated in real time using the cursor's movement delta, interpolated within canvas world coordinate space.
- **Delete Node:** Click the **red X** button on any node to remove it and all its connections. If the deleted node is a trigger, its paired Error Handler is also removed.

### 2.3 Auto Error Handler Pairing

When a trigger node is created, TG;BD automatically spawns an **Error Handler** node 220px below it and wires them together. This ensures every trigger has a fallback path for exception handling.

```
        ┌─────────────┐
        │  /start      │  ← Trigger node
        │  Command     │
        └──────┬───────┘
               │ output_1
        ┌──────▼───────┐
        │ Error Handler │  ← Auto-created
        │ {error}       │
        └──────────────┘
```

The compiler uses the first **non-error-handler** connection as the main flow path and only activates the error handler when an exception occurs.

---

## 3. Viewport Navigation & Scale Control

### 3.1 Canvas Panning

| Method | Activation |
|:---|:---|
| **Middle-Click Drag** | Hold `Mouse_Button_3` and drag |
| **Space + Left-Click Drag** | Hold `Space` + `Mouse_Button_1` and drag |

The view matrix translates its 2D axes based on the mouse displacement vector, offering fluid infinite workspace navigation. Panning is disabled while the context menu is open.

### 3.2 Dynamic Scaling (Zooming)

Scaling modifies the projection matrix scale factor with the cursor position as the transformation pivot, preventing spatial drift.

| Method | Step Size |
|:---|:---|
| **Scroll Wheel** | 0.05x per step |
| **Ctrl + Scroll Wheel** | 0.05x per step |
| **Ctrl + Plus/Minus** | 0.10x per step |
| **Zoom Widget (+/- buttons)** | 0.10x per step |
| **Zoom Input (text field)** | Any value ≥ 0.01 |

> **Safe range (buttons & scroll):** 0.10x (10%) to 3.00x (300%). Manual text input allows any value ≥ 0.01.

---

## 4. Keyboard Combinations

### 4.1 Workspace Shortcuts

| Shortcut | Action |
|:---|:---|
| **Ctrl + Z** | Undo last action (node create/delete/move, connection, data change) |
| **Ctrl + Y** | Redo last undone action |
| **Ctrl + Shift + Z** | Redo (alternative) |
| **Delete** / **Backspace** | Delete selected node(s) |
| **Escape** | Close context menu / deselect nodes |

### 4.2 Viewport Shortcuts

| Shortcut | Action |
|:---|:---|
| **Ctrl + Plus** (`Ctrl+=`) | Zoom in by 0.10x |
| **Ctrl + Minus** (`Ctrl+-`) | Zoom out by 0.10x |
| **Ctrl + 0** | Reset zoom to 1.0x (100%) |
| **Ctrl + Scroll Wheel** | Zoom in/out by 0.05x steps |

### 4.3 Build & Test Shortcuts

| Shortcut | Action |
|:---|:---|
| **Ctrl + S** | Quick Compile/Download (uses stored token if available) |
| **Ctrl + T** | Quick Test (uses stored token if available) |

### 4.4 Node Editing Shortcuts

| Shortcut | Action |
|:---|:---|
| **Ctrl + C** | Copy selected node(s) |
| **Ctrl + V** | Paste copied node(s) at cursor position |
| **Ctrl + D** | Duplicate selected node(s) |

<details>
<summary><b>💡 Pro Tip: Rapid Bot Building</b></summary>

1. Right-click to spawn a trigger
2. Click the output circle and drag to create a wire
3. Press **Ctrl+D** to duplicate nodes with their config
4. Use **Ctrl+Scroll** to zoom out for the big picture
5. Press **Ctrl+T** to test instantly
</details>

---

## 5. UI Component Reference

| Component | Location | Behavior |
|:---|:---|:---|
| **Navbar** | Top (64px) | Logo on left; Clear Canvas, Live Test, Compile buttons on right |
| **Canvas** | Full center | Infinite grid, Drawflow-powered node graph |
| **Zoom Widget** | Bottom-right | `−` `[input]` `+` with progress bar |
| **Node Title Bar** | Top of each node | Category color dot + node name + red `✕` delete button |
| **Node Inputs** | Left side of node | Gray circles (sockets for incoming connections) |
| **Node Outputs** | Right side of node | Colored circles (sockets for outgoing connections) |
| **Wire** | Between nodes | Bezier curve, reroutable by dragging |

### 5.1 Node Color Coding

| Category | Color | Dot |
|:---|:---|:---:|
| Trigger | Red | 🔴 |
| Action | Blue | 🔵 |
| Logic | Green | 🟢 |
| Variable | Orange | 🟠 |
| System | Purple | 🟣 |
| Error | Gray | ⚪ |

### 5.2 Toast Notifications

Bottom-center popup notifications inform you of action results:
- **Success:** Green background, auto-dismisses after 2 seconds
- **Error:** Red background, persists until dismissed
- **Info:** Blue background, auto-dismisses after 3 seconds

---

## 6. Node Wiring & Connections

### Making a Connection

1. Hover over a node's **output circle** (right side)
2. Click and drag — a wire appears following the cursor
3. Release over another node's **input circle** (left side)
4. The connection is established with a bezier curve

### Connection Rules

| Constraint | Details |
|:---|:---|
| **One input per connection** | An input socket can only receive one wire |
| **Multiple outputs** | An output socket can connect to many inputs |
| **No circular connections** | The canvas prevents direct A→B→A loops |
| **Branching nodes** | `If Condition` and `Check Variable` have 2 outputs (true/false) |

### Removing a Connection

Click on any wire and press `Delete` or `Backspace`. The wire is removed without affecting the nodes.

---

## 7. Inline Keyboard Builder

Message-type nodes (`Send Message`, `Reply to Message`, `Send Multimedia`, `Edit Message Text`) include a **visual keyboard builder** in their configuration panel.

### How It Works

```
┌──────────────────────────────────┐
│  Inline Keyboard                 │
│  ─────────────────────           │
│  ┌────────┐ ┌─────────┐         │
│  │ Button │ │ Button  │  ← Row 1│
│  │  Text  │ │  Text   │         │
│  │ cb: x  │ │ cb: y   │         │
│  └────────┘ └─────────┘         │
│  ┌──────────────────┐           │
│  │ Button           │  ← Row 2  │
│  │  Text            │           │
│  │ cb: z            │           │
│  └──────────────────┘           │
│  [Add Row]  [Add Button]        │
└──────────────────────────────────┘
```

- **Each button** has a display `text` and a `callback_data` value
- **Each button row** can hold multiple buttons
- **Each button** adds a new output port (`output_2`, `output_3`, etc.) to the node
- **Only buttons with connections** generate callback handlers in the compiled output

---

## 8. Zoom & Viewport Controls

The zoom widget in the bottom-right corner provides:

| Control | Behavior |
|:---|:---|
| **− Button** | Decrease zoom by 0.10x (min 0.10x) |
| **+ Button** | Increase zoom by 0.10x (max 3.00x) |
| **Text Input** | Type any percentage (≥ 1%); press Enter or blur to apply |
| **Progress Bar** | Visual indicator of current zoom level within the 0.10x–3.00x range |

> The progress bar fills proportionally: 0% at 0.10x, 50% at 1.55x, 100% at 3.00x.

---

<p align="center">
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/%C2%AB_Previous-Getting_Started-blue?style=flat-square" alt="Previous"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/Next-%C2%BB_Node_Reference-blue?style=flat-square" alt="Next"></a>
</p>

<p align="center">
  <a href="./DocumentationPortal.md"><img src="https://img.shields.io/badge/Portal-1-blue?style=flat-square" alt="Portal"></a>
  <a href="./GettingStarted.md"><img src="https://img.shields.io/badge/1-Gray-darkgrey?style=flat-square" alt="Page 1"></a>
  <a href="./FirstMoves.md"><img src="https://img.shields.io/badge/2-Active-blue?style=flat-square" alt="Page 2"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/3-Gray-darkgrey?style=flat-square" alt="Page 3"></a>
  <a href="./CompilerArchitecture.md"><img src="https://img.shields.io/badge/4-Gray-darkgrey?style=flat-square" alt="Page 4"></a>
  <a href="./ApiIPC.md"><img src="https://img.shields.io/badge/5-Gray-darkgrey?style=flat-square" alt="Page 5"></a>
  <a href="./AdvancedWorkflows.md"><img src="https://img.shields.io/badge/6-Gray-darkgrey?style=flat-square" alt="Page 6"></a>
  <a href="./Troubleshooting.md"><img src="https://img.shields.io/badge/7-Gray-darkgrey?style=flat-square" alt="Page 7"></a>
  <a href="./NodeReference.md"><img src="https://img.shields.io/badge/Next-%C2%BB-blue?style=flat-square" alt="Next"></a>
</p>
