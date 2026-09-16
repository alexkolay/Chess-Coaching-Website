# Chess Coaching Platform

A simple booking site for chess lessons — coaches set their availability, students find a coach and request a time slot, the coach approves it and it's booked. Plain HTML/CSS/JS on the frontend, Node + Express + MongoDB on the backend. No framework, no build step, nothing fancy.

Still a work in progress and not live anywhere yet. The main flow — browse coaches, request a lesson, coach approves it — works end to end, but there are a few rough edges (see [Known Issues](#known-issues) below).

## Running it locally

You'll need Node and MongoDB installed.

1. `npm install`
2. Create a `.env` file in the project root:
   ```
   MONGODB_URI=mongodb://127.0.0.1:27017/chess-coaching
   JWT_SECRET=pick-something-random
   PORT=3001
   SEED_COACH_PASSWORD=whatever-you-want
   ```
3. Run `node server/seed.js` to create a starter coach account (check `server/seed.js` for the username), or skip this and just register your own accounts through the site.
4. `./start.sh` boots MongoDB and the server together, `./stop.sh` shuts both down.

Then open **http://localhost:3001**. (It's 3001 instead of 3000/5000 mainly because macOS's AirPlay Receiver likes to grab 5000.)

## How it's put together

Every page is a regular HTML file. They all load `api.js`, which is just a small wrapper around `fetch` that talks to the Express server and keeps track of your login token. There's no server-side rendering — the pages ask the API for data and fill in the page themselves, so you can open any `.html` file directly and it'll work against whatever server is running on port 3001.

```
public/     the site itself — HTML pages, api.js, styles
server/     the Express API and MongoDB models
tests/      backend tests
```

On the backend, everything lives in MongoDB — there are three collections (Users, Lessons, Schedules) and four groups of routes (`/api/auth`, `/api/users`, `/api/lessons`, `/api/schedules`). Logging in gets you a token that the browser holds onto and sends with every request after that. Coaches and students hit mostly the same routes but get different permissions — a coach can approve lesson requests and set their own availability, a student can request lessons and browse coaches, and the server checks this on every request rather than just hiding buttons in the UI.

Booking has some conflict checking built in too, so a coach or student can't accidentally end up double-booked for the same time.

## Testing

```bash
npm test
```

Runs the backend test suite (Jest + Supertest) against its own throwaway MongoDB database, so it won't touch any real data. It covers logging in/registering, booking conflicts, and the permission checks — things like a student not being able to approve their own lesson request, or a coach not being able to touch another coach's schedule.

## Known Issues

- Login tokens are stored in the browser's `localStorage`, which works but isn't the most secure approach — cookies would be a better long-term fix.
- Some pages could use a security pass before this goes anywhere public — nothing catastrophic, but a few spots trust user input more than they should.
- Not deployed anywhere yet, still local-only.

## If something's not working

- **Port already in use** — the app runs on 3001 instead of 5000 to dodge macOS's AirPlay.
- **MongoDB won't start** — check `~/mongodb/logs/mongod.log`.
- **Keeps logging you out / "not authorized"** — tokens expire after 7 days, just log back in.
- **Need to reset the starter coach account** — run `SEED_COACH_PASSWORD=... node server/seed.js` again.
