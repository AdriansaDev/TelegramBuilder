# 🖱️ First Moves & Core Architecture

**Telegram Builder (TG;BD)** utilizes a dual-input paradigm optimized for rapid canvas navigation, viewport transformations, and nodegraph orchestration. The design decouples visual state rendering from the underlying telemetry compilation.

### Input Mapping Specification

| Input Action | Target Component | Functional Outcome | Context / Constraints |
| :--- | :--- | :--- | :--- |
| **Right-Click** (`Mouse_Button_2`) | Canvas Workspace | Instantiates the contextual node creation menu. | Inhibited when hovering active node hitboxes. |
| **Left-Click** (`Mouse_Button_1`) | UI Elements / Nodes | Executes widget primitives, manages selection states, and handles node-to-node edge mapping. | Supports multi-selection via bounding box marquee dragging. |
| **Middle-Click Drag** (`Mouse_Button_3`) | Canvas Workspace | Translates the viewport camera matrix (Panning). | Active globally across all canvas coordinates. |
| **Scroll Wheel** (`Mouse_Wheel`) | Canvas Workspace | Modulates viewport scale factor (Zooming). | Anchored to the current screen-space cursor coordinate. |

---

### Workspace Initialization & Contextual Workflows

#### Node Instantiation Workflow
Triggering `Mouse_Button_2` over any vacant coordinate within the viewport subsystem dispatches an event to the UI manager, dynamically spawning the context-sensitive node creation menu. This interface acts as the primary gateway to the node registry (`Trigger`, `Message`, `Condition`, `Variable`, `Counter`, `Terminal`).

* **Coordinate Resolution:** The instantiation coordinates are mapped directly from viewport space to canvas world space.
* **State Management:** Opening the menu flags the canvas state as `Interaction_State: Interrupted` to prevent accidental panning during selection.

<img width="406" height="484" alt="ui_node_selector" src="https://github.com/user-attachments/assets/3910c912-615c-4091-8540-b5f5754c3251" />

---

#### Selection & Component Manipulation
Executing `Mouse_Button_1` triggers distinct interaction pipelines depending on the intersection of the screen-space cursor with the bounding boxes of entities present within the workspace.

* **Single Entity Selection:** A discrete click on an active node hitbox clears the previous selection registry and pushes the target entity's unique identifier to the focus stack, transitioning its visual state to `State: Selected`.
* **Marquee Selection (Bounding Box Drag):** Dragging the cursor while maintaining `Mouse_Button_1` pressure over an empty canvas coordinate instantiates a dynamic 2D marquee rectangle. The spatial indexer performs continuous intersection tests; all nodes whose position vectors fall within or cross the boundary of the marquee are atomically appended to the active selection array.
* **Node Translation (Dragging):** Maintaining `Mouse_Button_1` down over a selected node initializes coordinate synchronization. The entity's translation vectors are recalculated in real time using the cursor's movement delta, interpolated directly within the canvas world coordinate space.

---

#### Viewport Navigation & Scale Control
To facilitate adaptive navigation across extensive, multi-layered bot graphs, the rendering subsystem executes direct matrix transformations on the canvas camera.

* **Canvas Panning:** Activating `Mouse_Button_3` (or combining the `Space` modifier key with a `Mouse_Button_1` drag) shifts the system into `Interaction_State: Panning`. The view matrix translates its 2D axes based on the mouse displacement vector, offering fluid, infinite workspace navigation.
* **Dynamic Scaling (Zooming):** Scrolling the mouse wheel modifies the projection matrix scale factor logarithmically. This transformation uses the current screen-space cursor coordinate as the transformation pivot, preventing spatial drift and keeping the operator's visual focus intact during magnification changes.

## ⌨️ Keyboard Combinations

(not added yet.)
