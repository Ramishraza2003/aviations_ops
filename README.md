# Aviation Ops — SQLite-backed flight operations dashboard

A local aviation operations control board: live flight schedule, delays,
incident/error log, and fleet status — all read from and written to a real
SQLite database on disk (`db/aviation.db`), via Express + `better-sqlite3`.

## What's in here

```
aviation-ops/
├── server.js              Express app entry point
├── db/
│   ├── schema.sql          Table definitions (airports, aircraft, crew, flights, incidents, maintenance_logs)
│   ├── connection.js       Shared SQLite connection, applies schema on boot
│   ├── seed.js             Wipes and repopulates the database with sample data
│   └── aviation.db         Created automatically the first time you run the app or seed
├── routes/
│   ├── flights.js          list/detail/update flights
│   ├── incidents.js        list/create/resolve incidents
│   ├── aircraft.js         fleet status + maintenance history
│   ├── airports.js         airport lookup
│   └── stats.js            dashboard summary numbers
├── public/index.html       Frontend — flight board, incident log, fleet panel
└── package.json
```

## Run it

```bash
cd aviation-ops
npm install
npm run seed     # creates db/aviation.db and fills it with sample flights/incidents
npm start
```

Open **http://localhost:5000**.

No external database, no signup, no `.env` needed — SQLite is just a file
(`db/aviation.db`) that better-sqlite3 reads and writes directly on your
machine.

## What's real vs. what's static

- **Flights, incidents, aircraft, crew, airports** — all live in SQLite and
  are fetched over the API (`/api/flights`, `/api/incidents`,
  `/api/aircraft`, `/api/airports`, `/api/stats/summary`).
- **Updating a flight's status** (from the flight detail modal) writes an
  `UPDATE` to the `flights` table immediately.
- **Reporting an incident** inserts a new row into `incidents` and shows up
  in the log right away — try it from the "Report incident" button.
- **Resolving an incident** sets `resolved = 1` and stamps `resolved_at`.
- The dashboard **auto-refreshes every 20 seconds** so changes (including
  ones you make yourself) reflect without a manual reload.

## Resetting the data

```bash
npm run seed
```

This deletes `db/aviation.db` and rebuilds it from scratch with fresh
sample flights timed relative to whenever you run the seed — so the board
always looks "live" no matter when you open it.

## Schema at a glance

- `airports` — IATA code, name, city, country, timezone
- `aircraft` — tail number, model, status (`in_service` / `maintenance` / `grounded`), maintenance dates
- `crew` — name, role, license number, status
- `flights` — flight number, airline, aircraft, origin/destination, scheduled + actual times, gate, status
- `flight_crew` — join table linking flights to crew members and their role on that flight
- `incidents` — type, severity, description, linked flight (optional), resolved state
- `maintenance_logs` — service history per aircraft

## Notes

- `better-sqlite3` is a native module — `npm install` will download a
  prebuilt binary for your platform automatically in almost all cases. If
  it ever fails to install on your machine, the usual fix is installing the
  "Desktop development with C++" workload via Visual Studio Build Tools on
  Windows, but this is rarely needed.
- Flight status transitions aren't restricted by the current status (e.g.
  you can set a `landed` flight back to `scheduled`) — deliberately kept
  loose since this is an ops tool, not a strict state machine.
