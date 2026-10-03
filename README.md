#🎓 PrimeTechCollege_ERP

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![React](https://img.shields.io/badge/Frontend-React%2018%20%2B%20Vite-61dafb)
![PHP](https://img.shields.io/badge/API-PHP%208-777bb4)
![Node](https://img.shields.io/badge/Realtime-Node.js%20%2B%20Socket.IO-339933)
![MySQL](https://img.shields.io/badge/Database-MySQL%208-4479a1)

A full-stack campus management and community platform for colleges — combining a student social feed with the administrative machinery a campus actually runs on: admissions, fee collection, results and marksheets, timetables, attendance, transport and a suggestion box.

Built with a React + Vite frontend, a PHP REST API for the core modules, and a small Node.js/Socket.IO service for real-time chat and presence, all on a shared MySQL database.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [1. Database](#1-database)
  - [2. Environment variables](#2-environment-variables)
  - [3. PHP API](#3-php-api)
  - [4. Realtime server](#4-realtime-server)
  - [5. Frontend](#5-frontend)
- [Roles and Access](#roles-and-access)
- [API Overview](#api-overview)
- [Integrations](#integrations)
- [Building for Production](#building-for-production)
- [Security Notes](#security-notes)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Features

### Campus community
- **Social feed** — create posts, like, comment, trending topics, suggested people
- **Clubs & organizations** — browse and join campus clubs
- **Events** — campus events calendar
- **Chat** — real-time one-to-one messaging over Socket.IO, plus a dedicated faculty ↔ student channel
- **Notifications** — in-app notification centre
- **Study & resources** — shared study material and resource library
- **Profiles** — public student profiles with editable details

### Academics
- **Timetable** — per-course, per-semester timetable with an admin management panel
- **Results** — semester results with downloadable PDF marksheets (jsPDF)
- **Attendance** — faculty-entered attendance with an admin oversight panel
- **AI assistant** — a student-facing chat assistant backed by OpenRouter, with the student's course and semester injected as context

### Administration
- **Admissions** — online application flow with an enrolment fee payment step
- **Fee management** — fee structures, fee categories, receipts, and downloadable PDF receipts
- **Payments** — Razorpay payment buttons for semester fees and admission fees
- **Records management** — student and faculty CRUD, status changes, password resets
- **Suggestion box** — student submissions with an admin moderation panel
- **Transportation** — campus transport routes and details
- **Activity log** — admin actions recorded for audit
- **Telegram bot** — webhook integration for notifications and bot commands

### Public pages
- Landing page with campus tour and "Apply Now" modals
- Public fee structure page
- Rules & regulations page

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, Vite 5, React Router 6, CSS Modules |
| HTTP client | Axios |
| UI / utilities | lucide-react, date-fns, react-markdown |
| PDF generation | jsPDF, jspdf-autotable |
| Core API | PHP 8 (plain PHP REST endpoints, PDO) |
| Realtime | Node.js, Express 4, Socket.IO 4, mysql2 |
| Database | MySQL 8 |
| Payments | Razorpay Payment Buttons |
| AI | OpenRouter API |
| Messaging | Telegram Bot API |

---

## Architecture

<img width="7662" height="4175" alt="diagram" src="https://github.com/user-attachments/assets/8d647091-cc6c-4a46-90fe-ddf951a9515a" />

---

## Project Structure

```
college-campus-connect/
├── backend/
│   ├── api/                  # PHP REST endpoints
│   │   ├── router.php        # PATH_INFO shim for `php -S`
│   │   ├── auth.php          # login / register / me
│   │   ├── admin.php         # admin CRUD + activity log
│   │   ├── admission.php     # admissions workflow
│   │   ├── faculty.php       # faculty dashboard + attendance
│   │   ├── fees.php          # fee structures, receipts
│   │   ├── results.php       # results & marksheets
│   │   ├── timetable.php     # timetable CRUD
│   │   ├── posts.php         # social feed
│   │   ├── clubs.php         # clubs
│   │   ├── events.php        # events
│   │   ├── messages.php      # message history
│   │   ├── users.php         # profiles
│   │   ├── suggestions.php   # suggestion box
│   │   ├── transportation.php
│   │   ├── ai-assistant.php  # OpenRouter proxy
│   │   └── telegram_webhook.php
│   ├── config/               # db.php, helpers.php, telegram.php
│   ├── helpers/              # telegram_helper.php
│   ├── server.js             # Socket.IO realtime server
│   └── .env.example
│
├── database/                 # schema + seed SQL (see load order below)
│
├── frontend/
│   ├── src/
│   │   ├── components/       # layout, feed, widgets, modals
│   │   ├── contexts/         # AuthContext
│   │   ├── pages/            # route-level pages
│   │   │   └── admin/        # admin management panels
│   │   ├── utils/            # api.js, rbac.js, PDF generators
│   │   ├── config/           # razorpay.js
│   │   └── styles/
│   └── vite.config.js
│
├── LICENSE
├── SECURITY.md
└── README.md
```

---

## Getting Started

### Prerequisites

- **Node.js** 18 or newer
- **PHP** 8.0 or newer with `pdo_mysql` and `curl` enabled
- **MySQL** 8.0 or newer

### 1. Database

Create the schema and load the seed data. Run the base schema first, then the
module schemas, then the seeds:

```bash
# Base schema (creates the `college_campus` database)
mysql -u root -p < database/schema.sql

# Module schemas
mysql -u root -p college_campus < database/schema_admission.sql
mysql -u root -p college_campus < database/schema_faculty_modules.sql
mysql -u root -p college_campus < database/schema_faculty_student_messages.sql
mysql -u root -p college_campus < database/schema_fee_management.sql
mysql -u root -p college_campus < database/Schema_fee_categories_addon.sql
mysql -u root -p college_campus < database/schema_results.sql
mysql -u root -p college_campus < database/schema_results_upgrade.sql
mysql -u root -p college_campus < database/Schema_timetable.sql
mysql -u root -p college_campus < database/schema_suggestions.sql
mysql -u root -p college_campus < database/schema_transportation.sql
mysql -u root -p college_campus < database/schema_telegram.sql
mysql -u root -p college_campus < database/schema_telegram_v3.sql

# Patches / fixes
mysql -u root -p college_campus < database/schema_fix_missing_columns.sql
mysql -u root -p college_campus < database/schema_fee_structure_sync_fix.sql
mysql -u root -p college_campus < database/schema_admin_security_fix.sql

# Seed data
mysql -u root -p college_campus < database/seed.sql
mysql -u root -p college_campus < database/seed_admission.sql
mysql -u root -p college_campus < database/seed_faculty_modules.sql
mysql -u root -p college_campus < database/Seed_timetable.sql
```

Two optional scripts:

- `database/schema_static_admin_seed.sql` — inserts a placeholder `users` row so
  admin actions appear in the activity log. The app works fully without it.
- `database/fix_windows_setup.sql` — corrections for Windows/XAMPP setups.

### 2. Environment variables

```bash
cp backend/.env.example backend/.env
```

Then fill in `backend/.env`:

| Variable | Purpose |
| --- | --- |
| `PORT` | Port for the Node realtime server (default `3001`) |
| `CLIENT_URL` | Primary frontend origin, e.g. `http://localhost:5173` |
| `EXTRA_CLIENT_URLS` | Comma-separated additional allowed origins |
| `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASS`, `DB_NAME` | MySQL connection |
| `APP_TOKEN_SECRET` | Secret used to sign session tokens |
| `COLLEGE_NAME` | College name shown in generated documents |
| `OPENROUTER_API_KEY` | API key for the AI assistant |
| `OPENROUTER_MODEL` | Model slug passed to OpenRouter |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `TELEGRAM_BOT_USERNAME` | Telegram bot username |
| `TELEGRAM_WEBHOOK_SECRET` | Shared secret validating incoming webhooks |

Generate a token secret with:

```bash
php -r "echo bin2hex(random_bytes(32));"
```

`backend/.env` is gitignored. Never commit real credentials.

### 3. PHP API

```bash
cd backend
php -S localhost:8000 api/router.php
```

The router is required for the built-in server — it injects `PATH_INFO` so that
routes like `/api/admin.php/students` resolve correctly. On Apache use
`AcceptPathInfo On`; on Nginx, pass the path through to PHP-FPM as usual.

### 4. Realtime server

```bash
cd backend
npm install
npm run dev        # nodemon, or `npm start` for plain node
```

Runs on `http://localhost:3001` and handles chat messages, presence and live
notifications over Socket.IO.

### 5. Frontend

```bash
cd frontend
npm install
npm run dev
```

Open <http://localhost:5173>. Vite proxies `/api` to `http://localhost:8000`.

> **Note on API base URLs.** `frontend/src/utils/api.js` currently hardcodes the
> production host for both `PHP_BASE` and `NODE_BASE`. For local development,
> change both to `/api` so the Vite proxy handles them, or point them at
> `http://localhost:8000` and `http://localhost:3001` respectively.

---

## Roles and Access

Three roles, enforced client-side by `frontend/src/utils/rbac.js` and
server-side in the PHP endpoints.

| Role | Access |
| --- | --- |
| **Admin** | Superuser. Every module, full CRUD across students, faculty, fees, results, timetable, admissions, suggestions and records. |
| **Faculty** | Faculty dashboard, attendance entry, results entry, timetable, notices, own profile. |
| **Student** | Student dashboard, feed, clubs, events, chat, study, AI assistant, timetable, results, fees and receipts, placements, admissions, suggestion box, own profile. |

Admin sign-in is a separate route (`/admin`) and uses static credentials defined
in `backend/config/helpers.php` rather than a database user — see
[Security Notes](#security-notes).

---

## API Overview

All PHP endpoints live under `/api` and follow `/<script>.php/<resource>`.

| Endpoint | Responsibilities |
| --- | --- |
| `auth.php` | `login`, `register`, `me` |
| `admin.php` | Student/faculty records, status changes, activity log |
| `admission.php` | Applications, enrolment, admission fee |
| `faculty.php` | Faculty dashboard, attendance |
| `fees.php` | Fee structures, categories, receipts |
| `results.php` | Result entry and student result lookup |
| `timetable.php` | Timetable CRUD by course and semester |
| `posts.php` | Feed posts, likes, comments |
| `clubs.php` / `events.php` | Clubs and campus events |
| `messages.php` | Message history |
| `users.php` | Profiles, suggested people |
| `suggestions.php` | Suggestion box submit and moderate |
| `transportation.php` | Transport routes |
| `ai-assistant.php` | `POST /chat` — proxies to OpenRouter |
| `telegram_webhook.php` | Telegram bot webhook receiver |

Realtime endpoints on the Node server:

- `GET /api/messages/:convId` — message history for a conversation
- `GET /api/conversations/:userId` — a user's conversations
- Socket.IO events for sending messages, typing indicators and presence

Typed client wrappers for all of the above are exported from
`frontend/src/utils/api.js` (`authApi`, `adminApi`, `feesApi`, `resultsApi`,
`timetableApi`, and so on).

---

## Integrations

**Razorpay.** Payment Button IDs are centralised in
`frontend/src/config/razorpay.js`. To move from Test to Live mode, replace the
four IDs there with Live Mode IDs from the Razorpay dashboard — no other file
changes.

**OpenRouter.** The AI assistant proxies through `backend/api/ai-assistant.php`,
so the API key stays server-side and is never exposed to the browser. The proxy
caps conversation length, per-message size and total payload size, and enriches
requests with the student's course and semester.

**Telegram.** Configure `TELEGRAM_BOT_TOKEN` and `TELEGRAM_BOT_USERNAME`, then
point your bot's webhook at `https://<your-domain>/api/telegram_webhook.php`.
Incoming requests are validated against `TELEGRAM_WEBHOOK_SECRET`.

---

## Building for Production

```bash
cd frontend
npm run build      # outputs to frontend/dist/
```

Deploy `frontend/dist/` as static files, serve `backend/api/` through PHP-FPM,
and run `backend/server.js` under a process manager such as PM2 or systemd. The
`.htaccess` files in `backend/api/` and `frontend/dist/` handle SPA fallback and
API routing on Apache.

Set `CLIENT_URL` (and `EXTRA_CLIENT_URLS` if you serve both apex and `www`) to
your production origins so CORS and Socket.IO accept them.

---

## Security Notes

A few things worth knowing before deploying this publicly (the vulnerability-reporting
process and a full production checklist are in **[SECURITY.md](SECURITY.md)**):

- **Admin credentials are static.** `STATIC_ADMIN_EMAIL` and
  `STATIC_ADMIN_PASSWORD` are defined as constants in
  `backend/config/helpers.php`. Change them before deploying, and prefer moving
  them into `.env` alongside the other secrets.
- **Only the admin session is server-signed.** Student and faculty sessions are
  managed client-side in `AuthContext`, which is why `api.js` deliberately does
  not clear their session on a background `401`. Treat student/faculty
  authorisation as advisory on the client and enforce it in PHP.
- **`backend/.env` is gitignored** and must stay that way. Rotate any secret
  that has ever been committed.
- **`frontend/dist/` is gitignored** — build artifacts should not be versioned.

---

## Troubleshooting

**Logged in, but clicking a menu item bounces me to Login.**
Already handled — `api.js` only wipes the session on a `401` for admin users.
If it reappears, check that `ccc_user` in localStorage has the expected `role`.

**PHP endpoints return 404 or an empty `PATH_INFO`.**
You are almost certainly running `php -S` without the router. Start the server
with `php -S localhost:8000 api/router.php`.

**CORS errors from the frontend.**
Add your origin to `CLIENT_URL` or `EXTRA_CLIENT_URLS` in `backend/.env` and
restart both backends.

**Chat does not connect.**
Confirm the Node server is running on `PORT` and that `NODE_BASE` in
`frontend/src/utils/api.js` points at it.

**Admin actions work but do not appear in the activity log.**
Run `database/schema_static_admin_seed.sql`. The log insert fails its foreign
key check without that placeholder row; the action itself always succeeds.

---

## Contributing

Issues and pull requests are welcome. For larger changes please open an issue first, keep database
changes in clearly named SQL files, and never commit secrets (`backend/.env`, API keys, bot
tokens) or real student data.

---

## License

Released under the [MIT License](LICENSE) © 2026 Vaibhav Chauhan.
