# TaskFlow - Final App Architecture & Logic Blueprint

This document provides a complete and detailed specification of the TaskFlow application's architecture, data structures, state management, and UI logic. It is designed to serve as the definitive blueprint for rebuilding the application in a new technology stack (e.g., Flutter with Dart).

---

## 1. Data Storage & Persistence

The application employs a unified, platform-aware persistence strategy.

### 1.1. Core Persistence Layer (`src/hooks/usePersistentState.ts`)

- **Mechanism**: A custom hook `usePersistentState` is the single source of truth for all data persistence.
- **Platform-Awareness**:
    - On native mobile platforms (iOS/Android via Capacitor), it uses the `@capacitor/preferences` API, which writes to native `SharedPreferences` (Android) or `UserDefaults` (iOS).
    - In a web browser, it gracefully falls back to using `window.localStorage`.
- **Data Versioning**:
    - All data saved to storage is wrapped in an object that includes a version number: `{ version: 1, data: { ... } }`.
    - This allows for future data migrations if the data schema changes.

### 1.2. Data Models (`src/lib/types.ts`)

The entire application revolves around these core data structures:

- **`Author`**: Represents a user.
    - `name: string` - The user's display name.
    - `avatarUrl: string` - A URL or Base64 Data URI for the user's profile picture.

- **`TaskList`**: A category for tasks or journal posts.
    - `id: string` - A unique identifier (e.g., 'work', 'personal').
    - `name: string` - The display name of the list (e.g., "Work").
    - `color: string` - An RGB or HSL string representing the list's color.
    - `icon: string` - The string name of a `lucide-react` icon (e.g., 'Briefcase').

- **`Subtask`**: A checklist item within a `Task`.
    - `id: string` - A unique identifier.
    - `title: string` - The content of the subtask.
    - `isCompleted: boolean` - The completion status.

- **`Task`**: The primary data model for a to-do item.
    - `id: string` - Unique identifier.
    - `title: string` - The main title of the task.
    - `description?: string` - Optional detailed notes.
    - `listId: string` - The `id` of the `TaskList` it belongs to.
    - `isCompleted: boolean` - The primary completion status.
    - `isImportant: boolean` - Whether the task is marked as important (starred).
    - `isMyDay?: boolean` - If `true`, the task appears on the "My Day" view.
    - `myDaySetDate?: string` - An ISO date string marking when the task was added to "My Day". Used to identify "Leftover" tasks.
    - `isFixed?: boolean` - If `true`, the task has a fixed time and should not be moved by the auto-scheduler.
    - `dueDate?: string` - An optional due date in 'yyyy-MM-dd' format.
    - `startTime?: string` - An optional start time in 'HH:mm' format.
    - `duration?: number` - An optional duration in minutes.
    - `subtasks?: Subtask[]` - An array of subtasks.
    - `createdAt: string` - An ISO date string of when the task was created.

- **`JournalPost`**: A blog or journal entry.
    - `id: string` - Unique identifier.
    - `slug: string` - A URL-friendly version of the title.
    - `title: string` - The post's title.
    - `content: string` - The full content of the post, stored as an HTML string.
    - `excerpt: string` - A short summary of the post.
    - `coverImage: string | null` - A Base64 Data URI or a placeholder ID for the cover image.
    - `author: Author` - The author of the post.
    - `date: string` - The publication date string (e.g., '8/16/2023').
    - `readingTime: number` - Estimated reading time in minutes.
    - `listId: string` - The `id` of the `TaskList` it belongs to.

---

## 2. State Management

State is managed by two systems: `React Context` for global data and `Zustand` for UI-specific state.

### 2.1. Global Data Context (`src/context/AppContext.tsx`)

This context provider manages the core data of the application. It uses `usePersistentState` to load and save its state.

- **State & Actions**:
    - `tasks: Task[]`: The array of all tasks.
        - `setTasks(tasks: Task[])`: Replaces the entire task array. Used by the auto-scheduler.
        - `addTask(task: Task)`: Adds a new task.
        - `updateTask(taskId, updates)`: Updates specific fields of an existing task.
        - `deleteTask(taskId)`: Removes a task.
    - `journalPosts: JournalPost[]`: The array of all journal posts.
        - `addJournalPost(post)`
        - `deleteJournalPost(postId)`
    - `lists: TaskList[]`: The array of all user-created lists.
        - `addList(list)`
    - `currentUser: Author`: The currently logged-in user's profile.
        - `setCurrentUser(user)`

### 2.2. UI Theme Store (`src/store/useThemeStore.ts`)

A lightweight `Zustand` store manages all UI appearance settings to decouple it from core app data. This state is also persisted using `zustand/middleware/persist`.

- **State & Actions**:
    - `colorTheme: string`: The name of the active color theme (e.g., 'Default', 'Orange').
    - `mode: 'light' | 'dark' | 'system'`: The current appearance mode.
    - `uiSize: number`: An index (0-4) corresponding to UI sizes ['XS', 'S', 'M', 'L', 'XL'].
    - `cardStyle: 'glass' | 'default' | 'flat' | 'bordered'`: The visual style of cards.
    - Actions: `setColorTheme`, `setMode`, `setUiSize`, `setCardStyle`.

---

## 3. Pages and UI Logic

This section details the purpose and key interactions of each page.

### 3.1. Authentication (`/login`, `/register`, `/set-profile`)

- **`login`**: A simple form. On submission, it checks if `currentUser` exists in storage. If yes, navigates to `/today`. If not, alerts the user to sign up.
- **`register`**: A simple form. On submission, it simulates registration and navigates to `/set-profile`.
- **`set-profile`**:
    - **Function**: Allows a new user to set their `name` and `avatarUrl`.
    - **Avatar Selection**: Provides three methods via tabs:
        1.  **Random**: Displays 5 randomly generated avatars from `dicebear.com`. A "Refresh" button generates a new set.
        2.  **Generate (AI)**: An input field takes a text prompt. A "Go" button calls the `generateAvatar` AI flow. While generating, a particle animation loader is shown. The result is a new SVG avatar.
        3.  **Upload**: A button triggers a hidden file input. The user selects an image, which is read as a Base64 Data URI.
    - **Action**: The "Save and Continue" button saves the `name` and selected `avatarUrl` to the `currentUser` state and navigates to `/today`.

### 3.2. Main Layout (`/(main)/layout.tsx` & `BottomNavBar.tsx`)

- **Layout**: A container with a max-width for a phone-like view. It renders the main content and the `BottomNavBar`.
- **`BottomNavBar`**:
    - **Navigation**: Provides links to the five main pages: Today, Inbox, Calendar, Journal, Settings. The active link has a styled indicator.
    - **FAB (Floating Action Button)**: A prominent "Plus" button.
        - **Interaction**: Clicking it toggles a "speed dial" menu.
        - **Menu Items**: The menu reveals two options: "Add Task" and "Add Journal", each with an icon and label, which navigate to their respective pages. Animations are used for the FAB rotation and menu item emergence.

### 3.3. My Day Page (`/today`)

- **Purpose**: The main daily dashboard, presented as a **Timeline View**.
- **Header**: Displays the current user's avatar, "My Day" title, and the current date.
- **"AI Plan" Button**:
    - **Function**: Triggers the `autoScheduleTasks` logic from `src/lib/task-scheduler.ts`.
    - **Logic**: This function takes all tasks marked `isMyDay`, respects `isFixed` tasks, avoids predefined break times (lunch, dinner), and calculates an optimal `startTime` for each flexible task based on importance, due date, and duration. It then updates the entire `tasks` array with the new times.
    - **UI**: Has a shimmering, gradient "aurora" style. Shows a loader animation when scheduling is in progress.
- **"Leftover" Group**:
    - **Logic**: At the top, it displays tasks where `isMyDay` is true, but `myDaySetDate` is from a previous day. This is a list of unfinished tasks from yesterday.
    - **Actions**: A "..." menu provides bulk actions: "Move all to Today" (updates `myDaySetDate`), "Complete all", "Remove all from My Day" (`isMyDay: false`).
- **Timeline View**:
    - **Data Source**: This is the core view. It processes tasks that are `isMyDay` and have a `startTime`. The `buildTimelineItems` utility (`src/lib/timeline-utils.ts`) is used to inject "coffee breaks" and "gaps" between timed tasks.
    - **`TimelineItem.tsx`**: The component responsible for rendering the timeline visuals.
        - **Time Marker**: Shows the `startTime` (e.g., "10:00").
        - **Visuals**: Renders a vertical line with a central dot. The dot's style changes based on task state:
            - **`isCompleted`**: Solid fill with a checkmark.
            - **`isFixed`**: Dashed border.
            - **`isOverdue`**: Red border.
        - **Duration**: If `task.duration` exists, the vertical line below the dot is rendered thicker and its height is proportional to the duration, creating a "time block".
    - **`TaskCard.tsx`**: Rendered inside `TimelineItem`. It's a compact version of the task card.
- **"Done" Group**: At the bottom, displays all `isMyDay` tasks that are `isCompleted`, in a simple list.

### 3.4. Inbox Page (`/inbox`)

- **Purpose**: The master list of all tasks.
- **Header**: Title and a "Filter" button.
- **Filter/Sort Popover**: The filter button opens a popover with extensive options, all of which are persisted to storage.
    - **Filter by Status**: All, Incomplete, Completed.
    - **Filter by Importance**: All, Important, Not-Important.
    - **Sort By**: Default (by `startTime`), Due Date, Importance, Creation Date.
- **List Tabs**: A horizontal scrollable list of all `TaskList` items (plus an "All" button). Selecting a list filters the tasks shown.
- **Task Rendering**:
    - If sorted by "Default", tasks are grouped into "Expired", "Upcoming", and "Done" based on their `startTime` relative to the current time.
    - If sorted by any other criteria, tasks are shown in a single flat list.

### 3.5. Calendar Page (`/calendar`)

- **Purpose**: A monthly view of tasks.
- **Calendar Component**: A full-page calendar.
    - **Indicators**: Dates that have tasks display small dots underneath the number. The color and number of dots can indicate task count and importance.
- **Task List**: Below the calendar, a list displays all tasks scheduled for the currently selected date. Clicking a different date on the calendar updates this list.

### 3.6. Journal & Blog Pages (`/journal`, `/journal/[slug]`, `/journal/new`)

- **`journal`**: The main listing page for all `JournalPost` items.
    - **Features**: Has similar Search, Filter (by list), and Sort (by date, reading time) functionality as the Inbox page.
    - **`JournalCard.tsx`**: Each post is displayed as a card with its cover image, title, excerpt, and metadata.
- **`journal/new`**: A full-screen modal for creating a new post.
    - **Fields**: Cover image uploader (saves as Base64), `TaskList` selector, Title input, and Content `textarea`.
    - **Action**: "Save Post" button creates a new `JournalPost` object and adds it to the state.
- **`journal/[slug]`**: Displays a single post.
    - **Layout**: Shows cover image, title, author info, and renders the `content` HTML string.
    - **Actions**: "Delete" and "Share" buttons in the header.

### 3.7. Settings Pages (`/settings`, `/settings/appearance`, `/settings/profile`)

- **`settings`**: The main hub, providing navigation to sub-pages.
- **`settings/appearance`**:
    - **Function**: Controls all visual aspects of the app, using the `useThemeStore`.
    - **Controls**:
        - **Mode**: Tabs for Light, Dark, System.
        - **UI Size**: Tabs for XS, S, M, L, XL.
        - **Card Style**: Tabs for Glass, Default, Flat, Bordered.
        - **Theme Color**: A grid of color swatches for all available themes.
- **`settings/profile`**: A modal page identical in function to the initial `set-profile` page, allowing the user to update their name and avatar.

### 3.8. Modal Pages (`/add-task`, `/edit-task/[id]`, `/add-list`)

These pages appear as modals sliding up from the bottom.

- **`add-task` / `edit-task/[id]`**:
    - **Layout**: A vertically scrolling form for all `Task` properties.
    - **Controls**: Includes `Input` for title, `Textarea` for description, `Switch` for booleans (`isImportant`, `isFixed`, `isMyDay`), `Popover` with a `Calendar` for `dueDate`, `Input type="time"` for `startTime`, and a dynamic list for managing `subtasks`.
    - **Action**: "Save Task" / "Save Changes" button. The edit page also includes a "Delete" button.
- **`add-list`**:
    - **Layout**: A form to create a new `TaskList`.
    - **Controls**: `Input` for name, a grid of color swatches, and a grid of `lucide-react` icons to choose from.
    - **Action**: "Save List" button.

---

## 4. AI & Core Algorithms

### 4.1. AI Flows (`src/ai/flows/`)

- **`generate-avatar`**:
    - **Trigger**: Called from `set-profile` page.
    - **Logic**: Takes a string prompt. Instructs the LLM to generate an SVG string for a simple, vector-style avatar. Includes a fallback step to re-prompt and extract the `<svg>` tag if the first response includes extra text.
- **`prioritize-tasks`**:
    - **(Not Currently Integrated in UI)**
    - **Logic**: Takes an array of tasks. Instructs the LLM to analyze them and return the same array with an updated `isImportant` flag and a `reason` string explaining its decision.
- **`smart-schedule-suggestion`**:
    - **(Not Currently Integrated in UI)**
    - **Logic**: Intended for suggesting a single task's time. Takes task details, user's calendar data (as a string), and user habits (as a string). Instructs the LLM to return an optimal ISO 8601 start time and a `reasoning` string.

### 4.2. Task Scheduler Algorithm (`src/lib/task-scheduler.ts`)

- **`autoScheduleTasks`**:
    - **Purpose**: The "AI Plan" engine. It's a deterministic (non-AI) algorithm.
    - **Input**: The full array of `tasks` and a `ScheduleRule` object.
    - **Process**:
        1.  Separates tasks into "fixed" and "flexible".
        2.  Creates a list of "blocked slots" for the day, including breaks, fixed tasks, and times outside working hours.
        3.  Sorts flexible tasks using a heuristic score (importance, due date proximity, age).
        4.  Iterates through the sorted flexible tasks. For each one, it calls `findNextAvailableTime` to find the earliest slot that fits the task's duration without overlapping blocked slots.
        5.  Assigns the found `startTime` to the task and adds the task's new time block (plus a buffer) to the list of blocked slots.
    - **Output**: Returns a new `tasks` array with `startTime` assigned to the flexible tasks.
