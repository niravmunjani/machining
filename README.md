# NIDC Machining Department

Shop floor scheduling, production tracking and machine monitoring for the
machining department.

## What is here

```
public/    the web apps - this folder is what gets served
bridge/    the program that reads the Haas machines
sql/       database schema and maintenance scripts
tools/     helper scripts (speed up, offline bundle)
docs/      notes
```

### public/

| File | What it is |
|---|---|
| `index.html` | Sends phones to the mobile app and computers to the full one. |
| `scheduler.html` | Main app. Shift scheduling, production, OEE, quality, paint. |
| `mobile.html` | Phone and tablet version. Same data, built for a thumb. |
| `install-card.html` | Printable QR card so the crew can add the app to a phone. |
| `diagnose.html` | Connection test. Open it when something will not load. |

These are single files with no build step. Open one in a browser, or drop it
on any static host.

### bridge/

A small Node program that asks each Haas machine what it is doing every few
seconds and writes the result to the cloud.

**It must run on a computer on the shop network.** The machines are at
`192.168.1.x`, which only exists inside the building, so a cloud server
cannot reach them.

See `bridge/RUN-ON-ANY-SERVER.md`. Windows, Linux, Raspberry Pi and Docker
are all covered.

### sql/

Run these in the Supabase dashboard under SQL Editor.

| File | When |
|---|---|
| `FIX-RLS.sql` | App says "failed to load" or CORS errors |
| `0-ONE-QUERY.sql` | Quick health check |
| `1-DIAGNOSE.sql` | Fuller health check |
| `2-FIX.sql` | Reclaim space if tables are bloated |
| `4-ANALYZE.sql` | Gather statistics |
| `casting-schema.sql` | Casting MES tables |

## How it fits together

```
Haas machines  --MTConnect-->  bridge (shop PC)  --internet-->  Supabase
                                                                   |
                                             web app  <------------+
                                             mobile app <----------+
```

Everything reads and writes the same database, so an entry on a phone shows
on the shop PC within seconds.

## Hosting

These are static files. **No Docker, no build, no server.**

**Render** - `render.yaml` in this repo does it for you. Create a service from
this repo and Render reads that file. If you created it as a Web Service by
mistake it will look for a Dockerfile and fail: delete it and create a
**Static Site** instead, or just let `render.yaml` take over.

**Netlify** - drag the `public` folder onto app.netlify.com/drop, or connect
the repo and set the publish directory to `public`.

**Anything else** - serve the `public` folder. That is all there is to it.

Short links, set up in `render.yaml`:

| Path | Opens |
|---|---|
| `/` | picks phone or desktop automatically |
| `/m` | mobile, always |
| `/pc` | desktop, always |
| `/test` | connection test |

## Setup

1. Run `sql/casting-schema.sql` and `sql/FIX-RLS.sql` in Supabase
2. Copy `bridge/.env.example` to `bridge/.env` and put your machine IPs in
3. Start the bridge: `RUN-FOREVER.bat` on Windows, or `install-linux.sh`
4. Host `apps/scheduler.html` and `apps/mobile.html` anywhere static

## SECURITY - read before making this repo public

The app files contain a Supabase **anon key**. That key is meant to be public,
BUT every table currently has a wide open policy:

```sql
CREATE POLICY ... FOR ALL USING (true) WITH CHECK (true)
```

So the key gives **full read and write to the whole database**. Anyone who
found it could delete every schedule, job and production record.

**Keep this repository private.** For an internal tool that is the right
answer and costs nothing.

If it ever needs to be public, the key has to come out of the files and the
policies have to be tightened first.
