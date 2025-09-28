# TaskFlow Flutter Development Plan (Revision 3)

This document provides a highly-detailed, "pixel-perfect" plan to build the TaskFlow application. This revision focuses on deeply integrating the visual style and animation choreography from the reference materials (`ref` folder) with the specific data models and application logic defined in `plan.md`.

---

## Phase 1 & 2: Foundation & State Management

_(These phases remain unchanged as they form the necessary backend and architectural backbone for the detailed UI work to come.)_

---

## Phase 3: Feature Implementation - Core Task Management

### Step 3.1: Authentication Flow
_(Unchanged)_

### Step 3.2: "My Day" Timeline View (Pixel-Perfect Revision)

This section is completely revised to serve as the definitive blueprint for the "My Day" screen UI.

#### 3.2.1: The Timeline Canvas (`timeline_view.dart`)

This will be a stateful widget that builds the entire scrollable timeline.

- **Layout:** It will use a `Stack` to layer the timeline painter in the background and a `ListView.builder` (or `CustomScrollView`) for the tasks.
- **Background Painter (`timeline_painter.dart`):**
    - A `CustomPainter` will be responsible for drawing the non-interactive elements.
    - **Vertical Line:** A single, centered, 1px wide, light gray line will be drawn for the entire height of the view.
    - **Hour Markers:** For each hour in the workday (e.g., 7 AM to 9 PM), a text label (e.g., "9 AM") will be rendered to the *left* of the vertical line. The font will be small and gray.
    - **Current Time Indicator:** A 1px high, solid red horizontal line will be drawn across the entire width of the view, positioned based on the current system time. It will have a small, solid red circle (4px diameter) where it intersects the vertical timeline. This indicator must update in real-time without requiring a full screen rebuild.

#### 3.2.2: The Task Card Widget (`task_card.dart`)

This is the central UI element, representing a single `Task`. It will be a stateful widget to manage its own complex animations.

- **Card Styling:**
    - The card's general style will be driven by the `cardStyle` setting from your `theme_provider.dart`. For example, `'default'` will have a soft drop shadow, `'bordered'` will have a 1px outline, and `'flat'` will have neither.
    - It will have rounded corners (approx. 8-12px radius).

- **Positioning & Sizing:**
    - **Vertical Position:** The card's top edge will be precisely aligned with its `Task.startTime` on the timeline.
    - **Height:** The card's height will be directly proportional to its `Task.duration` in minutes. A mapping will be established (e.g., 1 minute = 2 logical pixels) to calculate the exact height. If `duration` is null, it will have a fixed, default height.

- **Internal Layout (Pixel-Perfect Mapping to `plan.md`):**
    - **Checkbox (Right Side):** A circular checkbox on the far right. Tapping this toggles the `Task.isCompleted` boolean and triggers the completion animation (see Phase 6).
    - **Color Indicator (Left Side):** A small, solid circle (approx. 8px diameter) will be displayed on the far left. Its color will be directly bound to the `color` property of the `TaskList` associated with the `Task.listId`.
    - **Text Content (Center):**
        - **Title:** The `Task.title` will be the primary text, with a bold font weight.
        - **Time & Icon:** Below the title, in a smaller, lighter gray font, will be a row containing:
            - The formatted time string (e.g., "10:00 AM - 11:30 AM"), derived from `Task.startTime` and `Task.duration`.
            - The icon corresponding to the `TaskList.icon` string, rendered using a package like `lucide_flutter`.
    - **Subtasks:** If `Task.subtasks` exist, a small indicator (e.g., "3/5") will appear below the time string.

#### 3.2.3: Layout Logic

- **Non-Timed Tasks:** The "Leftover" tasks (from `plan.md`) will be displayed in a separate, non-timeline list at the very top of the scroll view.
- **Completed Tasks:** The "Done" group will be a simple list at the bottom, showing only the title and a checkmark for completed tasks. When a task is completed via the animation, it is removed from the timeline and added to this list.
- **Overlapping Tasks:** The layout logic must handle tasks with overlapping times. When an overlap is detected, the cards will be rendered side-by-side, each taking up 50% of the available width to avoid visual collision.

### Step 3.3: Task & List Modals
_(Unchanged)_

---

## Phase 4 & 5: Secondary Views & AI

_(These phases remain unchanged.)_

---

## Phase 6: Finalization (Animation Choreography Revision)

### Step 6.1: Polish & Animation Choreography

This section is revised to be an explicit set of animation instructions.

#### 6.1.1: Task Completion Animation Choreography

This is a multi-stage, ~500ms animation triggered when the `TaskCard` checkbox is tapped. It will be managed by an `AnimationController` within the `task_card.dart` widget.

1.  **Stage 1 (0-150ms): Checkbox Fill.** The circular checkbox border animates to fill with the app's primary theme color, and a checkmark icon fades in.
2.  **Stage 2 (50-350ms): Title Strikethrough.** A strikethrough line animates across the `Task.title` text, drawing from left to right.
3.  **Stage 3 (200-500ms): Coordinated Fade & Shrink.**
    - The entire card's opacity is animated from 1.0 to 0.0.
    - *Simultaneously*, the card's height is animated from its full height to 0.
4.  **Stage 4 (Post-Animation): List Update.** Once the animation is complete, the parent `timeline_view.dart` will use an `AnimatedList` to smoothly animate the remaining tasks, closing the gap left by the removed item.

#### 6.1.2: Drag & Drop Interaction Choreography

This interaction allows re-scheduling of tasks directly on the timeline.

1.  **Activation:** A long press on a `TaskCard` initiates the drag state. This interaction will be **programmatically disabled** if the `Task.isFixed` property from `plan.md` is `true`.
2.  **Lift State:** On long press, the card's elevation (shadow) increases, and it may scale up slightly (e.g., to 1.05x size) to indicate it is "lifted."
3.  **Drag State:** As the user drags the card vertically, its position updates in real-time. Other tasks in the list animate to make space for it.
4.  **Drop State:** On release, the card animates smoothly into its new position, its elevation returns to normal, and its `Task.startTime` is updated in the application's state via the `TaskProvider`.

### Step 6.2 & 6.3: Testing & Deployment
_(Unchanged)_
