# Multiplayer Tetris — Backend Guide

## What the app is

This is a Tetris game that supports both singleplayer and multiplayer. Players can start a solo game immediately from the home screen, or they can host/join a room to play against others in real time. There's also a leaderboard where scores are tracked.

The frontend handles everything you'd see and interact with — the falling pieces, the board, the controls, the animations. Your job as the backend is to handle the things that need to happen *between* players or need to be stored somewhere permanently.

---

## The two ways the frontend talks to you

**REST API** — standard HTTP requests for things that don't need to be instant. The frontend calls an endpoint, waits for a response, and moves on. Used for creating rooms, joining rooms, and the leaderboard.

**WebSocket** — a persistent, always-open connection between each player and the server. Used for everything that needs to happen in real time during a game: syncing the piece sequence, broadcasting opponent boards, and managing room events as they happen.

---

## REST endpoints you need to build

**Room management**

When a player wants to host, the frontend asks the backend to create a room. The backend generates a short room code (something like `X7KQ`) and returns it so the host can share it with friends.

When another player wants to join, they submit that room code along with their chosen display name. The backend needs to validate that the room exists and isn't already full or mid-game, then confirm they've been added.

**Leaderboard**

The frontend needs to be able to fetch the current leaderboard — a ranked list of high scores — to display it (likely on the start screen or after a game ends).

After a game ends, the frontend submits the player's final score to the backend to be saved.

---

## WebSocket events you need to handle

Once a room is created and players are connected, the WebSocket connection takes over. Think of it as a chat room where you're the moderator passing messages between players. Here's every event that needs to work:

**Room lifecycle**

- When a player connects to a room's WebSocket, broadcast to everyone else in the room that a new player has joined, including their name.
- When a player disconnects, broadcast to everyone that they've left.
- The host gets a special status. When the host decides to start the game, the backend receives that signal and tells all players to begin.
- When the game ends (either everyone finishes or the last player standing wins), broadcast the final result to all players in the room.

**Piece sequence seeding**

This is one of the most important jobs. All players in a multiplayer game need to receive the same pieces in the same order — that's how the game stays fair. When a game starts, the backend generates a single random seed number and sends it to every player in the room simultaneously. Each client uses that seed to generate its own local piece queue. You're not controlling the pieces after this point — you're just making sure everyone starts from the same random starting point.

**Opponent board snapshots**

During the game, each player's client periodically sends a snapshot of their current board state — essentially a picture of how their grid looks right now. Your job is to receive that snapshot and forward it to every *other* player in the room. You're acting as a relay. The frontend will display these as small read-only previews of what opponents are doing.

**Lock-event payloads (passive cheat validation)**

Every time a piece locks into place on a player's board, their client sends you a small payload describing what just happened — which piece, where it landed, what the board looks like now. You receive these and can check whether they seem legitimate (does this match what should be possible given the seed and prior events?). Critically, the game never *waits* for your response here. The client has already moved on. This is purely a background audit — flag suspicious behaviour, but never hold up gameplay.

---

## Summary of everything the backend owns

| Concern | How |
|---|---|
| Generate and store room codes | REST |
| Validate room join requests | REST |
| Fetch leaderboard | REST |
| Save scores | REST |
| Broadcast player joined / left | WebSocket |
| Signal game start to all players | WebSocket |
| Distribute shared piece-sequence seed | WebSocket |
| Relay opponent board snapshots | WebSocket |
| Receive and passively validate lock events | WebSocket |
| Broadcast game-over and final rankings | WebSocket |

---

## A few things worth knowing before you start

**You are never the authority on gameplay.** The frontend runs the full game loop on its own — gravity, movement, scoring, line clears. You don't simulate the game. You distribute the starting seed, relay information between players, and record results at the end.

**Room state lives on the server.** You need to track which players are in which room, who the host is, and whether the game is in progress or in the waiting room. When a player disconnects mid-game you need to decide how to handle it (remove them from the room, notify others, potentially end the game if only one remains).

**The WebSocket connection is per-player but scoped to a room.** When you receive a message from one player, you usually need to know which room they belong to so you can forward it to the right people.

**Timing matters for the seed.** The piece-sequence seed must be sent to all players at the same moment — when the game starts. If one player gets it a second before another, their piece queues will diverge at some point because the randomiser runs at different rates. Send it as part of the single "game start" broadcast so everyone begins from the same state.
