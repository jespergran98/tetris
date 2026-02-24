I'm making a simple multiplayer tetris game in React with TypeScript featuring a singleplayer and multiplayer mode following modern 2026 React standards. The backend will be made in another repository, your goal now is to structure the file and folder structure of the React front-end. Your task is now to update the file and folder structure of the front-end. The game will feature a start screen where the user can select singleplayer or multiplayer. If the user selects multiplayer, they can select host or join, and a room code is made if the user selects host, and up to 10 people can join. During multiplayer gameplay, you can see your own screen in large, with the other screens around your tetris game with "ghosts" of the other player boards. A score/leaderboard is also tracked through the backend. The project follows clean folder structure standards. Use CSS variables from global.css for all styling. The website must be fully responsive with a clean, modern design and simple logic. Always provide complete file contents for any modified or new files. Provide your suggested update to this file and folder structure:

Current project structure:

projectroot/
├── node_modules/
├── public/
│   └── assets/
│
├── src/
│   ├── components/
│   ├── Global.css
│   └── main.jsx
│
├── .gitignore
├── eslint.config.js
├── index.html
├── package-lock.json
├── package.json
├── README.md
└── vite.config.ts

---

projectroot/
├── public/
│   └── assets/
│       └── fonts/
│
├── src/
│   ├── assets/
│   │   └── icons/
│   │
│   ├── components/
│   │   ├── ui/
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   └── Button.module.css
│   │   │   ├── Modal/
│   │   │   │   ├── Modal.tsx
│   │   │   │   └── Modal.module.css
│   │   │   ├── RoomCode/
│   │   │   │   ├── RoomCode.tsx
│   │   │   │   └── RoomCode.module.css
│   │   │   ├── ErrorBoundary/
│   │   │   │   └── ErrorBoundary.tsx
│   │   │   └── index.ts
│   │   │
│   │   ├── game/
│   │   │   ├── TetrisBoard/
│   │   │   │   ├── TetrisBoard.tsx
│   │   │   │   └── TetrisBoard.module.css
│   │   │   ├── GhostBoard/
│   │   │   │   ├── GhostBoard.tsx
│   │   │   │   └── GhostBoard.module.css
│   │   │   ├── NextPiece/
│   │   │   │   ├── NextPiece.tsx
│   │   │   │   └── NextPiece.module.css
│   │   │   ├── HoldPiece/
│   │   │   │   ├── HoldPiece.tsx
│   │   │   │   └── HoldPiece.module.css
│   │   │   ├── ScorePanel/
│   │   │   │   ├── ScorePanel.tsx
│   │   │   │   └── ScorePanel.module.css
│   │   │   ├── Leaderboard/
│   │   │   │   ├── Leaderboard.tsx
│   │   │   │   └── Leaderboard.module.css
│   │   │   └── index.ts
│   │   │
│   │   ├── layout/
│   │   │   ├── SingleplayerLayout/
│   │   │   │   ├── SingleplayerLayout.tsx
│   │   │   │   └── SingleplayerLayout.module.css
│   │   │   ├── MultiplayerLayout/
│   │   │   │   ├── MultiplayerLayout.tsx
│   │   │   │   └── MultiplayerLayout.module.css
│   │   │   └── index.ts
│   │   │
│   │   └── index.ts
│   │
│   ├── pages/
│   │   ├── StartScreen/
│   │   │   ├── StartScreen.tsx
│   │   │   └── StartScreen.module.css
│   │   ├── LobbyScreen/
│   │   │   ├── LobbyScreen.tsx
│   │   │   └── LobbyScreen.module.css
│   │   ├── GameScreen/
│   │   │   ├── GameScreen.tsx
│   │   │   └── GameScreen.module.css
│   │   ├── MultiplayerGameScreen/
│   │   │   ├── MultiplayerGameScreen.tsx
│   │   │   └── MultiplayerGameScreen.module.css
│   │   ├── LeaderboardScreen/
│   │   │   ├── LeaderboardScreen.tsx
│   │   │   └── LeaderboardScreen.module.css
│   │   ├── NotFound/
│   │   │   ├── NotFound.tsx
│   │   │   └── NotFound.module.css
│   │   └── index.ts
│   │
│   ├── router/
│   │   └── index.tsx
│   │
│   ├── hooks/
│   │   ├── useTetrisGame.ts
│   │   ├── useGameLoop.ts
│   │   ├── useKeyboard.ts
│   │   └── useMultiplayer.ts
│   │
│   ├── context/
│   │   ├── GameContext.tsx
│   │   └── RoomContext.tsx
│   │
│   ├── services/
│   │   ├── websocket.service.ts
│   │   └── api.service.ts
│   │
│   ├── types/
│   │   ├── game.types.ts
│   │   ├── room.types.ts
│   │   └── api.types.ts
│   │
│   ├── utils/
│   │   ├── tetrominos.ts
│   │   ├── boardHelpers.ts
│   │   └── scoring.ts
│   │
│   ├── constants/
│   │   ├── game.constants.ts
│   │   └── websocket.constants.ts
│   │
│   ├── styles/
│   │   └── global.css
│   │
│   ├── App.tsx
│   ├── App.module.css
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