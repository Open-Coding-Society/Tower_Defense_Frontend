## Technical Design Overview

This project exemplifies a modular, object-oriented approach to frontend game development, as seen in `index.md`:

### Architecture Specifics

- **Object-Oriented JavaScript**: Core game logic is encapsulated in ES6 classes(`Game`, `Enemy`, `Tower`, `Projectile`, `Upgrades`, `Points`), promoting maintainability and extensibility.
- **Dynamic DOM Manipulation**: All game elements (towers, enemies, projectiles, UI overlays) are created, updated, and removed dynamically using JavaScript, allowing for real-time interactivity and visual feedback.
- **Modular Asset/Data Management**: Game configuration (tower stats, images, path points, constants) is managed in centralized arrays and objects, making it easy to update or extend game content.
- **Backend API Integration**: The game integrates with a backend Python API for persistent points tracking, using asynchronous fetch calls and a dedicated `Points` class for state management.
- **Separation of Concerns**: UI rendering, game logic, and configuration are clearly separated. CSS is scoped within the page, and all gameplay logic is handled in a single script module.
- **Responsive UI Components**: UI elements such as health bars, coin/point displays, and popups are modular and update in response to game state changes.
- **Extensible Upgrade System**: The `Upgrades` class and tower-level logic allow for scalable addition of new tower abilities and upgrade paths.

### Key Components in This Project

- **index.md**: Main entry point, containing all game logic, UI, and integration code.
- **assets/js/api/config.js**: Provides configuration for backend API endpoints and fetch options.
- **Dynamic HTML/CSS**: All game visuals and controls are rendered and styled at runtime, with minimal reliance on static HTML.

### Design Principles in Practice

- **Encapsulation**: Each game entity manages its own state and DOM representation.
- **Reusability**: Shared logic (e.g., projectiles, upgrades) is abstracted for use across multiple tower/enemy types.
- **Scalability**: Adding new towers, enemies, or features requires minimal changes to existing code due to modular design.
- **Developer Experience**: Clear separation of configuration, logic, and UI makes the codebase approachable for further development and educational use.

---
