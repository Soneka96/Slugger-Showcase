# Slugger

Stop writing throwaway scripts for every batch API job — seed databases, migrate data, fire webhooks, and test API sequences from one desktop app that tracks every run.

![Electron](https://img.shields.io/badge/Electron-47848F?logo=electron&logoColor=white) ![React](https://img.shields.io/badge/React-61DAFB?logo=react&logoColor=black) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white) ![SQLite](https://img.shields.io/badge/SQLite-003B57?logo=sqlite&logoColor=white) ![Vitest](https://img.shields.io/badge/Vitest-6E9F18?logo=vitest&logoColor=white)

Running HTTP jobs by hand means lost progress when auth expires mid-session, no record of which files succeeded or failed, and a restart from scratch every time something breaks. Slugger maps folders of JSON files to HTTP endpoints, runs them sequentially with automatic token refresh, and writes a timestamped Markdown report after every folder. Pause mid-run and resume from the exact file where you stopped — nothing is lost.
Redux Toolkit · Clean Architecture · JWT Ed25519 activation · 487 unit tests

**Get early access — it's free.**
Reply to sonekasenior@gmail.com with what you're building. I respond within 24 hours.
Free tier included — no credit card required.

---

## What it does

Pick a folder of `.json` files and point it at an HTTP endpoint. Slugger sends each file as a request — in order, one at a time — and marks every file green, amber, or red as results come in. Configure as many folders as you need per project; they run top to bottom.

Auth is handled automatically: configure a login endpoint and token path once, and Slugger fetches a token before the run starts, injects it into every request, and re-authenticates if a response signals expiry.

Every run is saved to history. Each folder produces a timestamped Markdown report with per-file results; an aggregate report rolls up the most recent result per file across all pause/resume segments. Open any report line directly in VS Code or Cursor.

---

## See it in action

Pause mid-run and resume from the exact file where you stopped — the sidebar shows which folders are done, which are in progress, and which haven't started:

![IDE mid-run — users folder complete (8/8), products folder paused mid-way (2/7), live log trail on the right](docs/screenshots/hero-mid-run-paused.png)

Nothing re-runs. Nothing is lost.

---

Every run is stored with its status, file count, and a link to the Markdown report. When a run stops on an error, Slugger auto-opens the failing file so you know immediately what to fix:

![History tab showing three runs — done, stopped, stopped — with Open Report buttons and error notification toast](docs/screenshots/history-tab.png)

The done/stopped/error badges and the auto-jump to the failing file remove the need to dig through logs manually.

---

Each folder maps to its own endpoint — method, path, and custom headers configured once. The URL preview below the path field shows the full request URL before you run anything:

![Folder editor Config tab — POST method selected, /posts endpoint, Content-Type header, full URL preview visible](docs/screenshots/folder-editor-config.png)

---

Every file's last response is stored and pretty-printed. Click any file to see exactly what the API returned — copy it, open it in your report, or compare it against the next run:

![product_01.json Last Response tab — 201 JSON body with id, title, price fields; full log trail on the right](docs/screenshots/file-last-response.png)

---

## Features

**Running jobs**
- Batch HTTP execution — hundreds of files, one endpoint per folder, sequential with per-file status dots
- Pause and resume — stop mid-run; resumes from the exact next file; nothing re-runs unnecessarily
- Run Missing — re-run only files that have not yet succeeded, skipping anything already green
- Run a single file — ▶ button on any row runs just that file immediately
- Request delay — configurable inter-request pause per project, stop-aware so it never blocks shutdown

**History and reports**
- Run history — every past run stored with file count, status badge, and link to the Markdown report
- Timestamped Markdown reports — written per folder and aggregated per run; shareable and archivable
- Segment reports — each pause/resume segment produces a `segment_N.md` with per-file response bodies
- Console log persistence — last run's progress restored from the database on app reopen

**Project and folder management**
- Unlimited projects and folders, drag to reorder
- Folder skip toggle — exclude a folder from full runs without deleting it
- Per-folder endpoint, method, and custom headers
- Missing folder detection — highlights path in red with a change button when a directory moves or is deleted
- Project export / import — share a project config as a `.slugger.json` file; missing paths flagged on import
- All Projects screen — searchable list of every project with folder count, dates, and delete

**Auth**
- Login endpoint, token path, header injection, expiry detection by status code or response body
- Automatic re-auth mid-run on token expiry

**Workflow**
- Per-file detail view — raw file content and last response; copy response to clipboard
- Open file or report line directly in VS Code or Cursor (`--goto` line precision)
- Native OS notifications — run complete, pause, error, and manual stop
- Keyboard shortcuts — F5 run/resume, Space pause, Esc stop
- Resizable panels — sidebar and console widths saved and restored per session

**App**
- Freemium licensing — free tier always available; pro JWT (Ed25519, audience-bound) upgrades without restart
- Dark / light theme, Retro / Cozy density, zoom 50–200 %, English / Portuguese

---

## User guide

### Creating a project

1. Launch Slugger.
2. Click **New Project**, enter a name, and press **Create**.
3. The project opens in the IDE.

### Adding folders

A **folder** maps a directory of `.json` files to an HTTP endpoint.

1. Click **Add Folder** in the sidebar.
2. A system dialog asks you to pick a directory. The folder appears in the sidebar.
3. Click the folder to open its editor.

### Configuring a folder

The folder editor has two tabs:

**Config tab**

| Field | Description |
|---|---|
| Folder path | Directory on disk. Click **Change** to pick a new path via the system dialog. |
| Endpoint path | Path appended to the project base URL, e.g. `/api/users` |
| Method | GET, POST, PUT, PATCH, or DELETE |
| Headers | Key/value pairs sent with every request from this folder |
| Notes | Freeform notes for this folder |

**Files tab**

Lists all `.json` files found in the folder. Click a row to open the file detail view (content + last response). The ▶ button runs that single file immediately. The tab shows a file count badge, or `!` if the folder path does not exist on disk.

If a folder path no longer exists, the Config tab highlights the path in red with a warning card. Use **Change** to point it at the correct directory.

### Running

Click **Run** in the right panel to start. Files are processed in alphabetical order across all folders (top to bottom). A timer appears above the buttons while a run is active.

- **Pause** — stops after the current request finishes. Progress is saved.
- **Stop** — same as Pause.
- **Resume** — continues from the file after the last one processed.
- **Restart** — starts the entire run from scratch, clearing all status dots.

Status dots appear next to each file: grey (pending), amber (running), green (done), red (error).

### Reordering folders

Drag folders in the sidebar to reorder them. If a run is paused, a confirmation dialog warns that reordering will reset the run to idle — confirm to proceed or cancel to keep your pause point.

### Run reports

After each folder completes (and again at the very end of the run) Slugger writes a timestamped Markdown file:

```
<userData>/Projects/<project-name>/runs/run_YYYY-MM-DD_HH-MM-SS.md
```

To open the folder: **Project Settings › General › Run Reports › Open Folder**.

### History tab

Click **History** (next to **Logs**) to see all past runs. Each row shows:

- Date and time
- Files processed
- Status badge: **done** (green) / **stopped** (amber) / **error** (red)
- **Open MD** — opens the report file in your default Markdown viewer
- **Delete** — removes the history entry (the report file on disk is kept)

**Clear All** removes all history entries at once.

### Authentication

Open **Project Settings › Authentication** to configure a login flow:

1. Set the login endpoint and credentials (JSON body).
2. Set the token path (dot-notation into the response body, e.g. `data.accessToken`).
3. Set the header name and prefix (e.g. `Authorization` / `Bearer`).
4. Optionally configure expiry detection by status code or response body value.

Slugger obtains a token before the run starts and injects it into every request. If a request triggers the expiry criteria it re-authenticates and retries automatically.

### Resizing panels

Hover over the border between the sidebar and the editor, or between the editor and the console panel — the border turns purple to indicate it is draggable. Drag left or right to resize the panel (minimum 160 px, maximum 400 px). Sizes are saved automatically and restored on next launch.

### Settings

Open the **☰** menu (top-right) to access:

| Setting | Options |
|---|---|
| Language | English / Portuguese |
| Theme | Dark / Light |
| Style | Retro (monospace, sharp) / Cozy (rounded, bubbly) |
| Zoom | 50 % – 200 % |
| Code Editor | Auto-detect / VS Code / Cursor |

---

## Tech stack

| | |
|---|---|
| **Stack** | Electron · React 18 · TypeScript · Vite |
| **State** | Redux Toolkit · Clean Architecture |
| **Storage** | SQLite (better-sqlite3) |
| **Auth / licensing** | JWT EdDSA (Ed25519) · jose |
| **Tests** | 487 unit tests · Vitest |

---

## Design decisions

**Why pause/resume instead of retry?**
Batch jobs often fail mid-run due to auth expiry or transient server errors — not bad data. A retry-from-scratch approach re-sends files that already succeeded, which is wasteful and sometimes harmful (duplicate inserts). Pause/resume stores the last-processed file index in the database, so the run continues from exactly where it stopped. Nothing re-runs unnecessarily.

**Why Ed25519 (EdDSA) over HMAC or RS256 for licensing?**
HMAC requires the secret on both sides — shipping it in the app means anyone can forge a license. RS256 works asymmetrically but the key is larger and signing is slower. Ed25519 gives asymmetric verification (public key only ships in the app), small key size, and fast verification. The private key never leaves the developer machine.

**Why SQLite over a remote store for run history?**
The app is fully offline by design. Users run it against internal APIs that may not have internet access. SQLite keeps history, reports, and project config local — no account, no sync, no data leaving the machine.

---

## Source

Source is private. Email sonekasenior@gmail.com for read access.
