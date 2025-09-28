# TaskFlow Flutter Development Plan

This document outlines the step-by-step plan to build the TaskFlow application using Flutter, based on the detailed blueprint provided in `plan.md`.

---

## Phase 1: Foundation & Core Architecture

### Step 1.1: Project Setup
- **Task**: Initialize a new Flutter project.
- **Actions**:
    - Run `flutter create taskflow`.
    - Clean up the default counter app code.
    - Set up the folder structure: `lib/models`, `lib/providers`, `lib/screens`, `lib/widgets`, `lib/services`.

### Step 1.2: Dependencies
- **Task**: Add essential packages to `pubspec.yaml`.
- **Packages**:
    - `provider`: For state management.
    - `shared_preferences`: For local data persistence.
    - `go_router`: For declarative navigation.
    - `google_fonts`: For custom typography.
    - `lucide_flutter` (or similar): For icons.
    - `uuid`: For generating unique IDs.

### Step 1.3: Data Models
- **Task**: Create Dart classes for all data models defined in `plan.md`.
- **Files**:
    - `lib/models/task.dart`: `Task`, `Subtask`.
    - `lib/models/journal_post.dart`: `JournalPost`, `Author`.
    - `lib/models/task_list.dart`: `TaskList`.
- **Details**: Implement `toJson` and `fromJson` methods for easy serialization.

### Step 1.4: Persistence Layer
- **Task**: Implement a service to handle saving and loading data from `shared_preferences`.
- **File**: `lib/services/persistence_service.dart`.
- **Logic**: Create a generic service that can handle different data types and mimics the platform-aware logic from the blueprint.

---

## Phase 2: State Management & UI Shell

### Step 2.1: State Management with Provider
- **Task**: Set up `ChangeNotifierProvider`s for the core application data.
- **Files**:
    - `lib/providers/task_provider.dart`: Manages `tasks`.
    - `lib/providers/journal_provider.dart`: Manages `journalPosts`.
    - `lib/providers/list_provider.dart`: Manages `taskLists`.
    - `lib/providers/user_provider.dart`: Manages `currentUser`.
    - `lib/providers/theme_provider.dart`: Manages UI theme settings.
- **Integration**: Wrap the `MaterialApp` with these providers in `lib/main.dart`.

### Step 2.2: Routing & Navigation Shell
- **Task**: Configure `go_router` and build the main layout with the bottom navigation bar.
- **Files**:
    - `lib/router.dart`: Define all application routes (`/`, `/today`, `/inbox`, etc.).
    - `lib/widgets/main_layout.dart`: The main screen wrapper containing the `BottomNavigationBar` and the FAB.
- **Details**: Implement the "speed dial" FAB for adding new tasks and journal entries.

---

## Phase 3: Feature Implementation - Core Task Management

### Step 3.1: Authentication Flow
- **Task**: Build the login, registration, and profile setup screens.
- **Screens**:
    - `lib/screens/login_screen.dart`.
    - `lib/screens/register_screen.dart`.
    - `lib/screens/set_profile_screen.dart`.
- **Logic**: Implement avatar selection (random, AI-generated, upload).

### Step 3.2: "My Day" Timeline View
- **Task**: Implement the main dashboard with the timeline. This is the most complex UI component.
- **Screen**: `lib/screens/today_screen.dart`.
- **Widgets**:
    - `lib/widgets/timeline_item.dart`: The core visual component for a single task on the timeline.
    - `lib/widgets/task_card.dart`: A compact task card for the timeline.
- **Logic**:
    - Implement `buildTimelineItems` utility in Dart.
    - Handle rendering of "Leftover" and "Done" groups.

### Step 3.3: Task & List Modals
- **Task**: Create the modal screens for adding and editing tasks and lists.
- **Screens**:
    - `lib/screens/edit_task_screen.dart`.
    - `lib/screens/add_list_screen.dart`.
- **Details**: These will be full-screen dialogs or modal bottom sheets, containing the forms as described in the blueprint.

---

## Phase 4: Feature Implementation - Secondary Views

### Step 4.1: Inbox Screen
- **Task**: Build the master task list with filtering and sorting.
- **Screen**: `lib/screens/inbox_screen.dart`.
- **Logic**: Implement the filter/sort popover and the horizontal task list tabs.

### Step 4.2: Calendar Screen
- **Task**: Implement the monthly calendar view.
- **Screen**: `lib/screens/calendar_screen.dart`.
- **Package**: Use a package like `table_calendar` to simplify the calendar UI.
- **Logic**: Display task indicators on dates and show a list of tasks for the selected day.

### Step 4.3: Journal Feature
- **Task**: Build the screens for the journal.
- **Screens**:
    - `lib/screens/journal_list_screen.dart`.
    - `lib/screens/journal_detail_screen.dart`.
    - `lib/screens/journal_new_screen.dart`.
- **Widgets**: `lib/widgets/journal_card.dart`.

---

## Phase 5: Settings & AI Integration

### Step 5.1: Settings Screens
- **Task**: Implement the settings area.
- **Screens**:
    - `lib/screens/settings_screen.dart`.
    - `lib/screens/appearance_settings_screen.dart`.
    - `lib/screens/profile_settings_screen.dart`.
- **Logic**: Connect the UI controls to the `ThemeProvider` to update the app's appearance in real-time.

### Step 5.2: AI & Task Scheduling
- **Task**: Port the core algorithms and integrate the AI features.
- **Files**:
    - `lib/services/task_scheduler.dart`: Port the `autoScheduleTasks` algorithm from TypeScript to Dart.
    - `lib/services/ai_service.dart`:
        - Integrate the `firebase_ai` package.
        - Implement the `generateAvatar` flow.
        - (Optional) Implement `prioritize-tasks` and `smart-schedule-suggestion`.
- **Integration**: Wire the "AI Plan" button on the "My Day" screen to the `autoScheduleTasks` function.

---

## Phase 6: Finalization

### Step 6.1: Polish & Animations
- **Task**: Add animations and refine the user experience.
- **Actions**:
    - Animate the FAB speed dial.
    - Add subtle animations to card appearances.
    - Ensure smooth transitions between screens.

### Step 6.2: Testing
- **Task**: Write unit and widget tests for critical components.
- **Focus Areas**:
    - **Unit Tests**: `task_scheduler.dart`, state providers.
    - **Widget Tests**: `TimelineItem`, `TaskCard`.

### Step 6.3: Deployment
- **Task**: Prepare the application for release.
- **Actions**:
    - Build for Android and iOS.
    - Test on physical devices.
