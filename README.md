# BuildSphere — Full-Stack System Architecture

An institutional weekly reporting platform built for **BVRIT Hyderabad College of Engineering for Women, CSE Department**. BuildSphere enables structured data capture across 17 operational modules, provides a live telemetry vault, and exports formatted PDF reports — all served from a single deployable Node.js process.

---

## System Architecture

The system enforces a strict three-tier separation. Each layer has a defined boundary and communicates only through the HTTP API contract.

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                             │
│  /FRONTEND/index.html  ·  /FRONTEND/login.html              │
│  /FRONTEND/app.js                                           │
│                                                             │
│  Vanilla HTML · Tailwind CDN · Vanilla JS                   │
│  Communicates exclusively via fetch() to /api/* endpoints   │
└──────────────────────────┬──────────────────────────────────┘
                           │  HTTP (REST)
┌──────────────────────────▼──────────────────────────────────┐
│                        BACKEND                              │
│  /BACKEND/server.js                                         │
│                                                             │
│  Node.js · Express 4 · Multer (file handling) · CORS        │
│  Serves static frontend, exposes REST API, manages uploads  │
│  Enforces global no-cache headers on all responses          │
└──────────────────────────┬──────────────────────────────────┘
                           │  In-Process (runtime object)
┌──────────────────────────▼──────────────────────────────────┐
│                        DATABASE                             │
│  /DATABASE/schema.txt                                       │
│                                                             │
│  NoSQL-style in-memory JSON store (17 dynamic arrays)       │
│  Physical file evidence persisted to /BACKEND/uploads/      │
│  Schema: { id, timestamp, ...dynamic fields, proofFile }    │
└─────────────────────────────────────────────────────────────┘
```

> **Note:** The database layer is an in-memory runtime object. All records are lost on server restart. For production persistence, replace with MongoDB or PostgreSQL.

---

## Tech Stack

**Frontend**
- HTML5, Vanilla JavaScript (ES6+)
- Tailwind CSS (CDN)
- GSAP 3 — entrance and field animations
- SweetAlert2 — modal dialogs and toast notifications
- html2pdf.js — client-side PDF generation
- Google Fonts: Inter, Oswald, Space Mono
- Font Awesome 6 — iconography

**Backend**
- Node.js (runtime)
- Express 4.18 — HTTP server and routing
- Multer 1.4 — `multipart/form-data` file interception and disk storage
- CORS — cross-origin request handling

**Database**
- In-process JavaScript object (NoSQL-style, 17 keyed arrays)
- Local disk storage for uploaded file evidence (`/BACKEND/uploads/`)

---

## Core Features

- **17-Module Dynamic Data Entry** — Structured forms for faculty, student, departmental, and institutional events are rendered dynamically from a single configuration map. Each module supports optional file evidence upload (PDF or image).
- **Live Telemetry Vault** — A searchable, real-time view of all committed records fetched from the server with aggressive cache-busting (timestamp-appended URLs + `no-store` headers) to guarantee data freshness.
- **One-Click PDF Compilation** — Aggregates all committed session data into a formatted A4 landscape report using `html2pdf.js`, rendered on the client from a hidden print-safe DOM node and exported without a server round-trip.

---

## Local Setup

**Prerequisites:** Node.js ≥ 18 and npm must be installed.

```bash
# 1. Clone the repository
git clone https://github.com/CodeWithRJ006/buildsphere.git
cd buildsphere

# 2. Install backend dependencies
cd BACKEND
npm install

# 3. Start the server
npm start
```

The server starts on **http://localhost:3000**.

The frontend is served automatically by Express as static files — no separate frontend server is required.

---

## API Reference

| Method   | Endpoint              | Description                                      |
|----------|-----------------------|--------------------------------------------------|
| `GET`    | `/api/report`         | Retrieve full in-memory database (all 17 modules)|
| `POST`   | `/api/report/:section`| Commit a new record to the specified module      |
| `DELETE` | `/api/purge`          | Wipe all in-memory records and delete all uploads|

**File uploads** are handled via `multipart/form-data`. The field name must be `proofDocument`. Files are stored in `/BACKEND/uploads/` and served statically at `/uploads/<filename>`.

---

## Project Structure

```
buildsphere/
├── FRONTEND/
│   ├── index.html       # Main application shell (SPA)
│   ├── login.html       # Authentication gate
│   └── app.js           # All client-side logic — themes, forms, vault, PDF
├── BACKEND/
│   ├── server.js        # Express server — API routes, static serving, Multer
│   ├── package.json     # Dependencies manifest
│   └── uploads/         # Runtime-created; stores uploaded file evidence
└── DATABASE/
    └── schema.txt       # Canonical schema documentation for the data model
```

---

## Deployment

The application is configured for **Render.com** single-service deployment. The server reads `process.env.PORT` when available and falls back to `3000` for local development. No additional configuration is required.

---

## Author

**RJ** — [github.com/CodeWithRJ006](https://github.com/CodeWithRJ006)  
License: ISC
