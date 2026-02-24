# Multiplayer Tetris — Front-End File & Folder Structure

I'm building a multiplayer Tetris game split across two repositories. The front-end is a React application. Your sole task is to design the optimal front-end file and folder structure for this project.

---

## Instructions for the AI

### Tree format

Return a complete file tree representing the **final state of the file system** — every file that will exist after your changes. Do not annotate files inline. Do not include ghost entries for deleted files. The tree should be clean and navigable on its own.

### Change Log

After the tree, list every changed file exactly once as a **bold heading** with a one-sentence justification beneath it. Follow strict tree order throughout — that is, list each changed file in the same top-to-bottom order it appears (or would have appeared) in the tree. Deleted files are listed at the position they occupied in the old tree.

Every change must fall into one of these categories, stated at the start of the justification:

- **Added** — a new file that did not previously exist.
- **Removed** — a file that has been deleted; include its old path.
- **Renamed** — a file renamed in place; state both old and new names.
- **Moved** — a file whose folder changed; state both old and new paths.

When a decision rests on an unstated assumption, write a one-sentence **Assumption:** note on the line immediately above the relevant Change Log entry.

### Before submitting

Verify every change against the Structural Principles. If a change would violate a principle, correct it or flag the deviation and justify it explicitly.

Check every added file against this test: *is this file directly necessitated by a stated requirement?* A file is speculative if its only justification is "this might be useful later" — remove it.

---

## Tech Stack

- **Framework** — React 19 with TypeScript 5
- **Bundler** — Vite
- **Routing** — React Router v7
- **Styling** — CSS Modules with CSS variables from `styles/global.css`
- **State** — React Context + hooks only (no Redux, no Zustand, no external state library)
- **Communication** — WebSocket (real-time game events) + REST (leaderboard, room management)
- **No external game libraries** — all Tetris logic is pure TypeScript in `utils/`

---

## Backend Responsibilities (out of scope for this repo)

**WebSocket (real-time):**
- Distributes the shared 7-bag piece sequence seed to all players at game start
- Receives lock-event payloads from each client for passive cheat validation (never in the critical path of gameplay)
- Redistributes opponent board snapshots to all players in the room
- Manages room lifecycle events (player joined, left, game started, game ended)

**REST API:**
- Creating and joining rooms
- Fetching and submitting leaderboard scores

---

## Front-End Responsibilities

- All rendering and UI
- Full client-side game loop authority — movement, gravity, rotation, collision detection, locking, line clears, and scoring never wait on the backend
- On each piece lock, sending a lock-event payload to the backend for passive validation
- Seeding the local 7-bag sequence from the value the backend distributes at game start
- Displaying opponent board snapshots as read-only previews
- Routing between screens
- Responsive layout across desktop and tablet viewports
- Persisting user settings (volume, keybinds) to `localStorage`

---

## Application Screens & Flow

```
Start Screen  [Settings button always visible in top-right corner]
├── → Singleplayer → Game Screen (singleplayer mode)
└── → Multiplayer
    └── Lobby Screen
        ├── Host → room code displayed → waiting room → Game Screen (multiplayer mode)
        └── Join → enter room code → enter name → waiting room → Game Screen (multiplayer mode)

Settings Modal (overlay, not a route — accessible from Start Screen and Pause overlay)
├── Audio tab — master volume slider
└── Controls tab — rebindable keybind list
```

**Start Screen** — Singleplayer and Multiplayer buttons. Settings icon in the top-right opens the Settings Modal.

**Lobby Screen** — The Host path displays the generated room code and a waiting room listing connected players; only the host can press Start. The Join path shows a room-code input followed by a name input, then the same waiting room view.

**Game Screen** — A single route rendered by `GameLayout`, which derives its visual mode from the number of players in `RoomContext`. See the Game Layout section below for the two arrangements.

**Settings Modal** — An overlay accessible from the Start Screen (via the settings icon) and from the Pause overlay (singleplayer only). Two tabs: Audio (master volume slider, 0–100) and Controls (rebindable action list). Both are persisted to `localStorage`.

---

## Game Layout

`GameLayout` is the single structural shell for the game screen. It reads player count from
`RoomContext`: a count of one triggers singleplayer mode; two or more triggers multiplayer
mode. No prop is needed — mode is derived, not passed.

Both modes render the same centred player panel: a compact, self-contained unit consisting
of the Hold piece stub (left), the main board (centre), and a right column containing the
Next piece queue, Score, Level, and Lines cleared. This panel is identical in both modes
and always occupies the visual centre of the screen.

**Singleplayer mode** — The player panel fills the screen. Escape opens a Pause overlay;
the Settings Modal is accessible from within it.

**Multiplayer mode** — The player panel remains centred. Opponent board thumbnails are
arranged in the remaining spaced to the left and right of the panel — mirroring the
Tetris 99 convention. A compact position indicator (e.g. 4 / 8) is appended to the
right column of the player panel rather than occupying a separate sidebar.

Components private to `GameLayout` and co-located inside its folder:

| Component        | Rendered in            |
|------------------|------------------------|
| `PauseOverlay`   | Singleplayer mode only |
| `OpponentBoard`  | Multiplayer mode only  |

**Game Over overlay** — Used in both modes (singleplayer shows a simple loss screen;
multiplayer shows final rankings). Because it has two importers it is a shared
`game/` component.

---

## Game Features

**Playfield** — 10 × 20 grid rendered as a CSS grid of cells.

**Tetrominoes** — Seven pieces (I, J, L, O, S, T, Z); unique colour per piece as a CSS variable.

**Movement & Rotation** — left/right shift, soft drop, hard drop, SRS rotation with wall kicks.

**Ghost piece** — Rendered inside `TetrisBoard`; not a separate component.

**Gravity** — `requestAnimationFrame` loop; fall interval decreases with level.

**Levels** — `Math.floor(linesCleared / 10) + 1`.

**Scoring** — 1 line: 100 × level · 2 lines: 300 × level · 3 lines: 500 × level · 4 lines: 800 × level.

**7-bag randomiser** — All 7 pieces once per cycle. In multiplayer, every client seeds from the value the backend distributes at game start.

**Hold piece** — Once per turn; locked until the current piece locks down.

**Lock down** — Locks when a piece can no longer fall; triggers a lock-event payload to the backend.

**Game over** — Triggered when a new piece cannot spawn.

---

## Settings System

- Volume and keybind state lives in `context/SettingsContext.tsx` and is persisted to `localStorage` on every change.
- `hooks/useSettings.ts` is the sole consumer-facing hook for settings state.
- `hooks/useInput.ts` reads the live keybind map from `useSettings` so rebinding takes effect immediately without a page reload.
- Default keybinds are defined once in `constants/keybinds.constants.ts`.
- **`localStorage` rule:** `context/SettingsContext.tsx` is the only file in the entire codebase permitted to read or write `localStorage`. No component, hook, util, service, or any other file may access it directly.

---

## Architecture: WebSocket Three-Layer Separation

1. **`services/websocket.service.ts`** — raw WebSocket class with no React dependency; handles connection lifecycle, serialisation, send/receive, and reconnection.
2. **`hooks/useWebSocket.ts`** — React hook wrapping the service; ties the connection to the component lifecycle; exposes connection status and a typed send function.
3. **`hooks/useMultiplayer.ts`** — game-level hook built on `useWebSocket`; translates incoming messages into room and opponent state; exposes the lock-event sender.

---

## Structural Principles

### 1 · Component layers

Components are organised into four layers. Place every component at the lowest layer that accurately describes its role:

- **`components/ui/`** — generic, domain-agnostic interface elements reusable across any project (buttons, modals, inputs, error boundaries).
- **`components/game/`** — Tetris-specific rendering components tightly coupled to game state (the board, piece previews, overlays).
- **`components/layout/`** — structural components that arrange other components within a screen without owning domain logic (`GameLayout` is the sole member).
- **`pages/`** — route-level entry points; one per screen in the application.

### 2 · Component visibility and co-location

A component is **shared** if it is imported by two or more parents. A component is **private** if it is imported by exactly one parent.

**Shared components** live in their own named folder at the appropriate layer: `ComponentName/ComponentName.tsx`. Include `ComponentName/ComponentName.module.css` alongside the `.tsx` file if and only if that component applies component-scoped styles; omit it for purely structural or behavioural components with no styles of their own.

**Private components** must be co-located inside their parent's folder as a flat sibling file — `SubName.tsx` and, if styled, `SubName.module.css` — rather than being given their own named subfolder. The one exception: if co-locating would bring the total number of files directly inside the parent folder (counting the parent's own `.tsx` and `.module.css`) above five, give the private component its own named subfolder *inside* the parent's folder (`ParentName/SubName/SubName.tsx`) rather than promoting it to the shared layer. A private component must never be imported from anywhere other than its direct parent.

> **Developer note (not an AI instruction):** if a private component later gains a second importer, move it to its own named folder at the appropriate shared layer at that time.

### 3 · Barrel files

`components/ui/`, `components/game/`, `components/layout/`, and `pages/` each export their **shared** members — every component that lives in its own named subfolder at that layer — via a barrel `index.ts`. Co-located private sub-components are never re-exported from barrels; they are internal to their parent and must not be imported through the barrel by any other file.

A *barrel* is a file that re-exports members from multiple sibling files or subfolders. A single-file module entry point such as `router/index.tsx` is not a barrel and is not subject to this rule.

No other folder — `context/`, `hooks/`, `utils/`, `services/`, `types/`, `constants/` — gets a barrel file.

### 4 · Hooks, utils, and services

- Hooks contain only React logic (state, effects, refs). All pure calculations — board manipulation, collision detection, scoring, rotation math, bag generation — belong in `utils/`.
- Services contain no React dependencies.

### 5 · Types

- Any type exported from its origin file must be defined in `types/`.
- Types used only within a single file and never exported may be defined locally in that file. Do not create a `types/` entry for a type that never leaves its origin.

### 6 · Constants

- No hardcoded strings or magic numbers in components or hooks — all such values live in `constants/`.
- Route paths are defined once in `constants/routes.constants.ts` and imported everywhere they are needed.
- Default keybinds are defined once in `constants/keybinds.constants.ts` and imported everywhere they are needed.

### 7 · Styles

All values that encode a design decision — colours, spacing, font sizes, timing, `z-index` values, border radii, opacity levels, and transition durations — must be CSS variables declared in `styles/global.css`. No raw values for any of these categories in any `.module.css` file.

### 8 · Naming

- `PascalCase` for components and types.
- `camelCase` for hooks, utils, services, and constants files.
- Tooling-generated and tooling-configured files (`vite-env.d.ts`, `vite.config.ts`, `eslint.config.js`, etc.) are exempt.

---

## Your Task

Review the current project structure below and return a fully improved version. You may add, remove, rename, or reorganise files and folders only where doing so is directly justified by a stated requirement or structural principle. Every change must make the structure leaner, clearer, or more correct — not all three conditions are required, but at least one must be demonstrably true.

**Current project structure to improve:**

projectroot/
├── public/
│   └── fonts/
│
├── src/
│   ├── assets/
│   │   └── icons/
│   │
│   ├── components/
│   │   ├── game/
│   │   │   ├── TetrisBoard/
│   │   │   │   ├── TetrisBoard.tsx
│   │   │   │   ├── TetrisBoard.module.css
│   │   │   │   ├── BoardCell.tsx
│   │   │   │   └── BoardCell.module.css
│   │   │   ├── PiecePreview/
│   │   │   │   ├── PiecePreview.tsx
│   │   │   │   └── PiecePreview.module.css
│   │   │   ├── HoldPanel/
│   │   │   │   ├── HoldPanel.tsx
│   │   │   │   └── HoldPanel.module.css
│   │   │   ├── NextPanel/
│   │   │   │   ├── NextPanel.tsx
│   │   │   │   └── NextPanel.module.css
│   │   │   ├── ScorePanel/
│   │   │   │   ├── ScorePanel.tsx
│   │   │   │   └── ScorePanel.module.css
│   │   │   ├── GameOverOverlay/
│   │   │   │   ├── GameOverOverlay.tsx
│   │   │   │   └── GameOverOverlay.module.css
│   │   │   └── index.ts
│   │   │
│   │   ├── ui/
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   └── Button.module.css
│   │   │   ├── Modal/
│   │   │   │   ├── Modal.tsx
│   │   │   │   └── Modal.module.css
│   │   │   ├── SettingsModal/
│   │   │   │   ├── SettingsModal.tsx
│   │   │   │   ├── SettingsModal.module.css
│   │   │   │   ├── AudioTab.tsx
│   │   │   │   └── ControlsTab.tsx
│   │   │   ├── ErrorBoundary/
│   │   │   │   └── ErrorBoundary.tsx
│   │   │   └── index.ts
│   │   │
│   │   └── layout/
│   │       ├── GameLayout/
│   │       │   ├── GameLayout.tsx
│   │       │   ├── GameLayout.module.css
│   │       │   ├── PauseOverlay/
│   │       │   │   ├── PauseOverlay.tsx
│   │       │   │   └── PauseOverlay.module.css
│   │       │   ├── OpponentBoard/
│   │       │   │   ├── OpponentBoard.tsx
│   │       │   │   └── OpponentBoard.module.css
│   │       │   └── LeaderboardSidebar/
│   │       │       ├── LeaderboardSidebar.tsx
│   │       │       └── LeaderboardSidebar.module.css
│   │       └── index.ts
│   │
│   ├── pages/
│   │   ├── StartScreen/
│   │   │   ├── StartScreen.tsx
│   │   │   └── StartScreen.module.css
│   │   ├── LobbyScreen/
│   │   │   ├── LobbyScreen.tsx
│   │   │   ├── LobbyScreen.module.css
│   │   │   ├── RoomCode/
│   │   │   │   ├── RoomCode.tsx
│   │   │   │   └── RoomCode.module.css
│   │   │   └── PlayerList/
│   │   │       ├── PlayerList.tsx
│   │   │       └── PlayerList.module.css
│   │   ├── GameScreen/
│   │   │   ├── GameScreen.tsx
│   │   │   └── GameScreen.module.css
│   │   ├── NotFound/
│   │   │   ├── NotFound.tsx
│   │   │   └── NotFound.module.css
│   │   └── index.ts
│   │
│   ├── router/
│   │   └── index.tsx
│   │
│   ├── context/
│   │   ├── GameContext.tsx
│   │   ├── RoomContext.tsx
│   │   └── SettingsContext.tsx
│   │
│   ├── hooks/
│   │   ├── useGameLoop.ts
│   │   ├── useGameState.ts
│   │   ├── useHoldPiece.ts
│   │   ├── useInput.ts
│   │   ├── useLockDelay.ts
│   │   ├── useMultiplayer.ts
│   │   ├── usePieceQueue.ts
│   │   ├── useSettings.ts
│   │   └── useWebSocket.ts
│   │
│   ├── services/
│   │   ├── rest.service.ts
│   │   └── websocket.service.ts
│   │
│   ├── utils/
│   │   ├── bag.utils.ts
│   │   ├── board.utils.ts
│   │   ├── collision.utils.ts
│   │   ├── lock.utils.ts
│   │   ├── rotation.utils.ts
│   │   ├── scoring.utils.ts
│   │   └── tetromino.utils.ts
│   │
│   ├── types/
│   │   ├── api.types.ts
│   │   ├── game.types.ts
│   │   ├── room.types.ts
│   │   ├── settings.types.ts
│   │   └── websocket.types.ts
│   │
│   ├── constants/
│   │   ├── game.constants.ts
│   │   ├── keybinds.constants.ts
│   │   ├── routes.constants.ts
│   │   ├── tetromino.constants.ts
│   │   └── websocket.constants.ts
│   │
│   ├── styles/
│   │   └── global.css
│   │
│   ├── App.tsx
│   ├── main.tsx
│   └── vite-env.d.ts
│
├── .gitignore
├── eslint.config.js
├── index.html
├── package-lock.json
├── package.json
├── README.md
├── tsconfig.json
└── vite.config.ts

---

Read the attached guide and current file structure in full before making any changes. Follow every Structural Principle exactly — treat them as hard constraints, not suggestions. For every file you add, remove, rename, or move, annotate it in the tree and justify it in the Change Log. Do not add any file that cannot be traced directly to a stated requirement. Do not remove any file unless you can prove it is redundant or violates a principle. Every change must make the structure leaner, clearer, or more correct — not all three conditions are required, but at least one must be demonstrably true.