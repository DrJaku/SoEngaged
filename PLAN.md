# Wedding Planner App — Plan

TL;DR

Create a lightweight frontend-only Wedding Planner SPA (Vue 3 + Vite + Bootstrap) with core screens, a single-root JSON backup/export/import, and localStorage persistence. Build iteratively starting with the Dashboard.

Top-level features
- Screens: dashboard, to do list, notes, guest list, contacts and vendors, seating chart, budget, admin page.

Per-screen intended features
- Dashboard
  - Overview KPIs: tasks due (7d), RSVP %, budget summary, pending payments
  - Quick-add actions: guest, task, expense
  - Recent activity and import/export shortcuts
  - Links to detail screens
- To Do List
  - CRUD tasks with due date, priority, status; filters; bulk actions
- Notes
  - Simple notes with tags and search; export
- Guest List
  - Add/edit guests, RSVP tracking, plus-ones, dietary notes, import/export
- Contacts & Vendors
  - Store vendor details, link to budget items and contracts
- Seating Chart
  - Tables and seat assignments, printable layout, drag/drop (future)
- Budget
  - Line items, categories, paid/unpaid, totals and reports
- Admin
  - Backup (full JSON), restore, settings, migration helpers

Data model (root JSON for full backup/export/import)

Root shape (canonical):

```
{
  "meta": { "title": string, "createdAt": string, "updatedAt": string, "version": string },
  "dashboard": { "notes": string, "lastViewed": string },
  "tasks": [ { id, title, description, due, priority, status, createdAt } ],
  "notes": [ { id, title, content, tags, createdAt } ],
  "guests": [ { id, name, email, phone, rsvp, plusOne, dietary, group } ],
  "contacts": [ { id, name, role, cost, notes } ],
  "seating": { "tables": [ { id, name, capacity, seats: [ { seatId, guestId|null } ] } ] },
  "budget": [ { id, category, item, cost, paid, vendorId|null } ],
  "settings": { "currency", "locale", "autosave" },
  "activity": [ { id, action, payload, timestamp } ]
}
```

- localStorage key: `marryme-app-data` (migrate from `demo-app-data` when detected)
- Export filename example: `marryme-backup-20251222-153000.json`

Backup / import policies (decisions)
- Merge strategy: **file wins** (incoming file overrides same-id entries)
- Attachments: **metadata only** (no binary data embedded)
- Import options: Replace / Merge / Preview with summary counts

Dashboard — Acceptance Criteria
- Loads and renders with current data from `marryme-app-data`.
- Displays these KPIs with correct values: tasks due (next 7 days), RSVP % (responded yes / total invited), budget spent vs total, pending payments count.
- Quick-add creates an item (guest/task/expense), updates KPIs immediately, and persists to `marryme-app-data`.
- Import/export shortcuts open file dialog or trigger a download of the single-root JSON.
- Migration prompt appears if legacy `demo-app-data` exists, offering Merge / Replace / Ignore.

Next implementation steps (dashboard-focused)
1. Centralize persistence: create a small store module that reads/writes `marryme-app-data` and exposes get/set/merge utilities.
2. Scaffold `src/views/Dashboard.vue` and `src/components/KpiCard.vue`, `QuickAddModal.vue`.
3. Wire quick-add actions to the store and update KPIs reactively.
4. Add import/export actions in the UI (Admin area or header) that call the store's export/import helpers.
5. Iterate UI: add recent activity feed and links to detailed screens.

Notes / future
- Seating chart and drag/drop may require additional libraries (consider `interactjs` or SVG/canvas approach later).
- Keep the single-root JSON approach for simplicity; consider IndexedDB only if performance becomes a problem.

Backup decisions recorded
- Merge strategy: file-wins
- Attachments: metadata-only

Status
- Planning complete. Dashboard is next — ready to scaffold the view and store when you say go.
