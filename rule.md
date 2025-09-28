# TaskFlow Project Rules & Guidelines

This document establishes the strict coding standards, architectural principles, and design guidelines that must be adhered to throughout the development of the TaskFlow application. Its purpose is to ensure the final product is robust, maintainable, bug-free, and scalable.

---

### **1. Architecture: The Repository Pattern**

The application will follow a strict layered architecture to ensure a clear separation of concerns. This is paramount for future scalability, especially for migrating to a cloud backend.

- **UI Layer (Screens & Widgets):**
    - The UI's *only* responsibility is to display state and forward user events to the state management layer.
    - Widgets should be `StatelessWidget` wherever possible.
    - Widgets must **never** directly access data sources (e.g., SharedPreferences, API clients).
    - Widgets should remain "dumb" and simply reflect the state provided by the State Management layer.

- **State Management Layer (Providers):**
    - This layer, using `provider`, is the single source of truth for the UI.
    - It contains the application's view state and business logic (e.g., filtering lists, calculating "Leftover" tasks).
    - It is the only layer that can call the Service/Repository Layer.
    - All state modifications must happen through methods within a `ChangeNotifier`.

- **Service / Repository Layer (`lib/services`):**
    - This layer is the key to cloud migration. It abstracts the *source* of the data.
    - A `PersistenceService` (or `TaskRepository`) will be created. It will have a clean interface (e.g., `Future<List<Task>> getTasks()`, `Future<void> saveTask(Task task)`).
    - **Crucially**, the rest of the app will *only* interact with this interface, not the specific implementation.
    - The initial implementation will use `shared_preferences`. To migrate to the cloud, only the *internals* of this service will be changed (to use Firestore, etc.), and the rest of the application will function without modification.

- **Data Model Layer (`lib/models`):**
    - These are plain Dart objects representing the application's data (`Task`, `JournalPost`, etc.).
    - They must contain `fromJson` and `toJson` methods for serialization.
    - Models should be immutable where possible (all properties `final`). When an object needs to be updated, a `copyWith` method will be implemented to create a *new* instance with the updated values.

### **2. State Management Rules**

- **Unidirectional Data Flow:** Data flows in one direction: `Service Layer` -> `State Management Layer` -> `UI Layer`. Events flow in the opposite direction.
- **Provider Scoping:** Providers will be scoped to the lowest possible widget in the tree that requires them, to avoid unnecessary rebuilds.
- **Consumer & Selector:** Use `Consumer` widgets or `context.select` to rebuild only the specific widgets that depend on a piece of state, rather than the entire screen.

### **3. Code Style & Quality**

- **Formatting:** `dart format .` will be run after every significant code change. Code will not be committed unless it is fully formatted.
- **Linting:** Strict linting rules will be enforced. Any analysis errors or warnings must be addressed immediately. `flutter fix --apply .` will be used as a first step.
- **Naming Conventions:**
    - `UpperCamelCase` for classes, enums, and typedefs.
    - `lowerCamelCase` for methods, variables, and parameters.
    - `_lowerCamelCase` for private members.
    - File names will be `snake_case.dart`.
- **Modularity:** Create small, single-purpose widgets. If a `build` method becomes too nested or long, it must be broken down into smaller, reusable widgets.
- **Constants:** `const` will be used aggressively for constructors and widgets to improve performance by preventing unnecessary rebuilds.

### **4. UI & Design Principles**

- **Fidelity to the Plan:** The UI will be implemented to be a "pixel-perfect" representation of the designs and choreography detailed in `developStep.md`.
- **Responsiveness:** All UI components will be built to be responsive and adapt to different screen sizes and densities.
- **Theming:** All colors, fonts, and component styles will be defined in a central `ThemeData` object in `main.dart`. No hardcoded colors or styles will be used in individual widgets.

### **5. Error Handling & Asynchronous Operations**

- **Futures/Streams:** `FutureBuilder` and `StreamBuilder` will be used to handle the lifecycle of asynchronous operations in the UI (showing loading, error, and data states).
- **Robustness:** All methods in the Service Layer that perform I/O (disk access, network calls) must have `try-catch` blocks to handle potential exceptions gracefully.
- **User Feedback:** When an error occurs, the UI must display a clear, user-friendly message.
