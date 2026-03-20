# ArenaSync

ArenaSync is a real-time 1v1 arena backend built for portfolio use, prototyping, and the general human urge to overengineer a duel system at 2 AM.

It ships with:

- JWT auth
- matchmaking queue
- Socket.IO room flow
- server-authoritative movement/combat
- PostgreSQL persistence through Prisma
- Redis-backed queue fallback
- leaderboard and match history APIs
- replay/event logging

## Tech stack

- Node.js
- TypeScript
- Express
- Socket.IO
- Prisma
- PostgreSQL
- Redis
- Zod
- Pino

## Core idea

Two authenticated players join the queue.

The server matches them, creates a room, and becomes the source of truth for:

- position
- health
- attack cooldowns
- dash cooldowns
- winner resolution

The client only asks to perform actions.
The server decides whether those actions are legal.
That matters because trusting the client is how you end up with teleporting goblins and damage values from outer space.

## Quick start

### 1. Install dependencies

```bash
npm install
```

### 2. Copy env file

```bash
cp .env.example .env
```

### 3. Start Postgres + Redis

```bash
docker compose up -d
```

### 4. Generate Prisma client and run migration

```bash
npx prisma generate
npx prisma migrate dev --name init
```

### 5. Start dev server

```bash
npm run dev
```

Server runs on `http://localhost:4000` by default.

## REST API

### Auth

- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/auth/me`

### Users

- `GET /api/users/:id`
- `GET /api/users/:id/stats`

### Leaderboard

- `GET /api/leaderboard`

### Matches

- `GET /api/matches`
- `GET /api/matches/:id`
- `GET /api/matches/:id/replay`

## Socket events

### Client -> server

- `queue:join`
- `queue:leave`
- `player:move`
- `player:attack`
- `player:dash`
- `ping`

### Server -> client

- `connected`
- `queue:joined`
- `queue:left`
- `match:found`
- `match:start`
- `game:state`
- `player:hit`
- `player:dashed`
- `match:end`
- `pong`
- `error`

## Example socket client

```ts
import { io } from 'socket.io-client';

const socket = io('http://localhost:4000', {
  auth: {
    token: 'YOUR_JWT_TOKEN'
  }
});

socket.on('connect', () => {
  socket.emit('queue:join');
});

socket.on('match:start', () => {
  socket.emit('player:move', { dx: 1, dy: 0 });
  socket.emit('player:attack', { direction: 'right' });
});
```

## Match model

A match stores:

- both players
- winner
- status
- timing data
- replay events

Replay events include:

- room creation
- movement
- attack attempts
- hits
- dash events
- match finish

## What is intentionally simple right now

This is a strong MVP, not a full esport backend. So a few things are intentionally light:

- only 1v1 support
- single in-process game engine
- simple anti-cheat checks
- no reconnect recovery
- no spectator mode
- no horizontal scaling adapter for Socket.IO yet

## Good next upgrades

- ranked matchmaking bands
- reconnect flow
- spectating
- room expiration worker
- Redis Socket.IO adapter
- 2v2 support
- richer combat states like stun/block/iframes
- replay playback service

## Suggested demo flow for GitHub

1. Register two users
2. Open two socket clients
3. Join matchmaking on both
4. Move around and attack
5. Finish match
6. Check leaderboard and replay endpoint

## Project structure

```txt
src/
  config/
  lib/
  middleware/
  modules/
    auth/
    users/
    leaderboard/
    matches/
    matchmaking/
    game/
  app.ts
  server.ts
```

## Notes

This repo is opinionated on purpose. The code tries to feel like an actual person built it, which, astonishingly, should not be a rare feature. It is still clean enough to extend without wanting to punch the database.
