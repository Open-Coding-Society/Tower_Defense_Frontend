# 🏰 Tower Defense

A browser-based tower defense game inspired by **Clash Royale**, built using modern JavaScript (ES6). Players must defend their king by strategically placing and upgrading towers to stop waves of enemies. With real-time gameplay, an interactive UI, and persistent score tracking via API, this project delivers both fun and technical depth.

---

## ✨ Features

- 🧠 **Strategic Gameplay** – Build and upgrade various towers to fend off increasingly difficult enemies.
- ⚔️ **Enemy Waves** – Unique enemy types with distinct movement and health behaviors.
- 💰 **Resource System** – Earn coins and points for kills to unlock more upgrades.
- 🌐 **API Integration** – Tracks high scores and gameplay data across sessions.
- 🖱️ **Interactive UI** – Click-based controls for tower placement and upgrades.
- 🧩 **Modular Design** – Clean, scalable architecture using ES6 classes.

---

## 🚀 Getting Started

To run the game locally:

1. **Clone the repository**  
   ```bash
   git clone https://github.com/Open-Coding-Society/Tower_Defense_Frontend.git
   cd Tower_Defense_Frontend
   ```

2. **Start a local server**  
   You can use a simple HTTP server like:

   ```bash
   # Using Python 3
   python -m http.server
   ```

3. **Open your browser**  
   Visit `http://localhost:8000` (or the appropriate port) to start playing.

> 🔧 No build steps or external dependencies required—just open in the browser!

---

## 🎮 Usage

- Use the mouse to place towers on valid spots.
- Towers will automatically target enemies within range.
- Earn coins from defeated enemies to purchase more towers or upgrades.
- Defend your king—don’t let enemies reach the base!

---

## 🤝 Contributing

Contributions are welcome! Feel free to:

- Suggest new enemy or tower types
- Optimize gameplay or UI components
- Refactor core game logic

Please open an issue or fork and submit a pull request with clear descriptions.

---

## 🧭 Issues / Roadmap

We use **GitHub Issues** to:

- Log bugs or edge cases
- Propose and track new features
- Discuss architectural changes or refactors

---

## ⚙️ Technical Design Overview

This project exemplifies a modular, object-oriented approach to frontend game development, as seen in `index.md`:

### 🏗️ Architecture Specifics

- **Object-Oriented JavaScript**: Core game logic is encapsulated in ES6 classes (`Game`, `Enemy`, `Tower`, `Projectile`, `Upgrades`, `Points`), promoting maintainability and extensibility.
- **Dynamic DOM Manipulation**: All game elements (towers, enemies, projectiles, UI overlays) are created, updated, and removed dynamically using JavaScript, allowing for real-time interactivity and visual feedback.
- **Modular Asset/Data Management**: Game configuration (tower stats, images, path points, constants) is managed in centralized arrays and objects, making it easy to update or extend game content.
- **Backend API Integration**: The game integrates with a backend Python API for persistent points tracking, using asynchronous fetch calls and a dedicated `Points` class for state management.
- **Separation of Concerns**: UI rendering, game logic, and configuration are clearly separated. CSS is scoped within the page, and all gameplay logic is handled in a single script module.
- **Responsive UI Components**: UI elements such as health bars, coin/point displays, and popups are modular and update in response to game state changes.
- **Extensible Upgrade System**: The `Upgrades` class and tower-level logic allow for scalable addition of new tower abilities and upgrade paths.

### 🧩 Key Components in This Project

- `index.md`: Main entry point, containing all game logic, UI, and integration code.
- `assets/js/api/config.js`: Provides configuration for backend API endpoints and fetch options.
- **Dynamic HTML/CSS**: All game visuals and controls are rendered and styled at runtime, with minimal reliance on static HTML.

### 🧠 Design Principles in Practice

- **Encapsulation**: Each game entity manages its own state and DOM representation.
- **Reusability**: Shared logic (e.g., projectiles, upgrades) is abstracted for use across multiple tower/enemy types.
- **Scalability**: Adding new towers, enemies, or features requires minimal changes to existing code due to modular design.
- **Developer Experience**: Clear separation of configuration, logic, and UI makes the codebase approachable for further development and educational use.

---

## 🙌 Acknowledgments

- Inspired by the mechanics and visual style of **Clash Royale**
- Built by members of the **Open Coding Society**
- Thanks to open-source contributors and browser game developers for foundational tools and techniques
