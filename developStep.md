# TaskFlow Development Steps

This document is the master checklist and sequential plan for the entire TaskFlow application development. Each step must be completed and verified before proceeding to the next.

---

## **Phase 1: Foundation & Core Architecture**

### **Step 1.1: Project Setup**
- [ ] Initialize a new Flutter project.
- [ ] Delete the default `lib/main.dart` content.
- [ ] Create the primary directory structure inside `lib`:
    - `lib/models`: For data structures.
    - `lib/services`: For the data persistence/repository layer.
    - `lib/providers`: For state management (`ChangeNotifier`s).
    - `lib/screens`: For top-level page widgets.
    - `lib/widgets`: For reusable, smaller UI components.
    - `lib/core`: For routing, theming, and constants.

### **Step 1.2: Dependencies**
- [ ] Open `pubspec.yaml`.
- [ ] Add the following dependencies:
    - `provider`
    - `shared_preferences`
    - `go_router`
    - `google_fonts`
    - `lucide_flutter` (or a suitable icon pack)
    - `uuid`
- [ ] Run `flutter pub get` and ensure it completes without errors.

### **Step 1.3: Core Data Models**
- [ ] Create `lib/models/task.dart` and define the `Task` and `Subtask` classes with `final` properties.
- [ ] Create `lib/models/journal_post.dart` and define the `JournalPost` and `Author` classes.
- [ ] Create `lib/models/task_list.dart` and define the `TaskList` class.
- [ ] For each model, implement a `copyWith()` method.
- [ ] For each model, implement `fromJson(Map<String, dynamic> json)` factory constructor and `Map<String, dynamic> toJson()` method.

### **Step 1.4: Persistence Layer (Repository Pattern)**
- [ ] Create `lib/services/persistence_service.dart`.
- [ ] Define an abstract class `PersistenceRepository` with method signatures for all data operations (e.g., `Future<List<Task>> getTasks()`, `Future<void> saveTasks(List<Task> tasks)`).
- [ ] Create a concrete class `SharedPreferencesService` that implements `PersistenceRepository`.
- [ ] Implement all methods using the `shared_preferences` package, serializing/deserializing data using the model's `toJson`/`fromJson` methods.

---

## **Phase 2: State Management & UI Shell**

### **Step 2.1: State Management Providers**
- [ ] Create `lib/providers/task_provider.dart`. It will use the `PersistenceRepository` to load/save tasks and manage the in-memory state.
- [ ] Create providers for `journal_provider.dart`, `list_provider.dart`, and `user_provider.dart` following the same pattern.
- [ ] Create `lib/providers/theme_provider.dart` to manage UI appearance state (`ThemeMode`, `cardStyle`, etc.).

### **Step 2.2: Theming & Routing**
- [ ] Create `lib/core/theme.dart`. Define `lightTheme` and `darkTheme` `ThemeData` objects. All colors, fonts (`google_fonts`), and component themes (e.g., `elevatedButtonTheme`) will be defined here.
- [ ] Create `lib/core/router.dart`. Configure `GoRouter` with all the application routes as defined in `plan.md`.
- [ ] In `lib/main.dart`, set up the `MultiProvider` with all the created providers.
- [ ] Configure the `MaterialApp.router` to use the theme and router configuration.

### **Step 2.3: Main Layout Shell**
- [ ] Create `lib/widgets/main_layout.dart`.
- [ ] This widget will contain the `BottomNavigationBar` with links to Today, Inbox, Calendar, Journal, and Settings.
- [ ] Implement the main Floating Action Button (FAB) and the "speed dial" menu animation that reveals "Add Task" and "Add Journal" options.

---

## **Phase 3: Pixel-Perfect Feature Implementation**

### **Step 3.1: "My Day" Screen (Today Page)**
- [ ] Create `lib/screens/today_screen.dart`.
- [ ] **Timeline Painter:** Create `lib/widgets/timeline_painter.dart` as a `CustomPainter`. It will draw the vertical timeline, hour markers, and the real-time "current time" indicator line.
- [ ] **Task Card Widget:** Create `lib/widgets/task_card.dart`.
    - Implement the precise internal layout: colored `TaskList` circle on the left, title, time/icon row, and checkbox on the right.
    - The card's height must be calculated based on the `Task.duration`.
    - The card's styling must be derived from the `ThemeProvider`.
- [ ] **Screen Layout:**
    - Use a `Stack` to layer the painter and a `CustomScrollView` for the content.
    - Implement the logic to position task cards correctly along the timeline.
    - Implement the "Leftover" tasks section at the top.
    - Implement the "Done" tasks section at the bottom.

### **Step 3.2: Task & List Modals**
- [ ] Create `lib/screens/edit_task_screen.dart` as a modal page.
- [ ] Build the complete form for editing every property of a `Task`.
- [ ] Implement the "Save" and "Delete" logic, calling the appropriate methods in `TaskProvider`.
- [ ] Create and implement `lib/screens/add_list_screen.dart` similarly.

---

## **Phase 4: Secondary Screens**

### **Step 4.1: Inbox Screen**
- [ ] Create `lib/screens/inbox_screen.dart`.
- [ ] Implement the filter/sort popover.
- [ ] Implement the horizontally scrolling `TaskList` tabs.
- [ ] Display tasks grouped by "Expired," "Upcoming," and "Done," or as a flat list depending on the sort order.

### **Step 4.2: Calendar Screen**
- [ ] Create `lib/screens/calendar_screen.dart`.
- [ ] Integrate the `table_calendar` package.
- [ ] Implement the logic to display colored dots on days with tasks.
- [ ] Below the calendar, create the list view that shows tasks for the selected day.

### **Step 4.3: Journal Feature**
- [ ] Create `lib/screens/journal_list_screen.dart`, `journal_detail_screen.dart`, and `journal_new_screen.dart`.
- [ ] Implement the `JournalCard` widget for the list view.

---

## **Phase 5: Settings & AI**

### **Step 5.1: Settings**
- [ ] Create all settings screens (`settings_screen.dart`, `appearance_settings_screen.dart`, `profile_settings_screen.dart`).
- [ ] Connect the UI controls in the appearance screen to the methods in `ThemeProvider` to allow real-time theme changes.

### **Step 5.2: AI & Scheduler**
- [ ] Create `lib/services/task_scheduler.dart`. Port the deterministic `autoScheduleTasks` algorithm from TypeScript to Dart.
- [ ] Wire the "AI Plan" button in `today_screen.dart` to trigger this scheduling logic.
- [ ] (Optional Stretch Goal) Create `lib/services/ai_service.dart` and implement the `firebase_ai` features.

---

## **Phase 6: Finalization & Animation**

### **Step 6.1: Animation Choreography**
- [ ] **Task Completion:** In `task_card.dart`, implement the full animation sequence using an `AnimationController`:
    - 1. Checkbox fill.
    - 2. Title strikethrough.
    - 3. Coordinated opacity fade and height shrink.
- [ ] **List Animation:** In `today_screen.dart`, wrap the task list in an `AnimatedList` to handle smooth removal of completed tasks.
- [ ] **Drag & Drop:**
    - Implement `LongPressDraggable` and `DragTarget` for the task cards on the timeline.
    - Disable the drag functionality if `Task.isFixed` is true.
    - Ensure other tasks animate smoothly when a card is being dragged over them.

### **Step 6.2: Testing**
- [ ] Write unit tests for `task_scheduler.dart` and all `Provider` classes.
- [ ] Write widget tests for `task_card.dart`, verifying its layout and animation states.

### **Step 6.3: Deployment**
- [ ] Run the app on physical Android and iOS devices.
- [ ] Perform thorough end-to-end testing.
- [ ] Prepare for release.
