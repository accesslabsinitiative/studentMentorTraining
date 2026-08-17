# Curriculum Tracker (written by claude, change later)

An interactive website for tracking curriculum completion across five program
components: **Programming**, **CAD**, **Presentational Skills**, **Onboarding
Program**, and **Scenario Quest**.

Every member has an account. Students track and self-report their own
progress, some tasks are graded automatically via built-in quizzes, and the
program admin can see everyone's progress in one place. Reaching 100% overall
unlocks a downloadable (and optionally emailed) certificate of completion.

## Features

- **My Progress** (`/dashboard`) — a completion bar for each of the 5
  components plus one for total completion, with a checklist of tasks per
  component. Manual tasks are checked off by the student; tasks marked
  "auto-graded" are completed by passing a short built-in quiz — no
  self-reporting needed for those.
- **Updates** (`/updates`) — an announcements feed anyone can read; admins can
  post and delete updates.
- **Resources** (`/resources`) — links and materials grouped by component;
  admins can add/remove resources.
- **Certificate of completion** — once a student's total completion hits
  100%, a "Download certificate" button appears on their dashboard. The
  certificate (a generated PDF) is also emailed automatically if SMTP is
  configured (see below); otherwise it's just a download.
- **Admin view** (`/admin`) — every account with a per-component and total
  completion breakdown; click through to a student for their full task-level
  detail. Restricted to accounts with the `ADMIN` role.
- **Accounts** — email/password auth (NextAuth credentials provider,
  passwords hashed with bcrypt). The first person to register, and anyone
  whose email is listed in `ADMIN_EMAILS`, is automatically made an admin.

## Tech stack

- Next.js 16 (App Router) + React 19 + TypeScript
- Tailwind CSS 4
- Prisma ORM + SQLite (swap `DATABASE_URL` for Postgres/MySQL in production —
  no code changes needed beyond the Prisma `provider`)
- NextAuth v5 (credentials/password auth, JWT sessions)
- `pdf-lib` for certificate generation, `nodemailer` for optional email delivery

## Getting started

```bash
cp .env.example .env    # then edit ADMIN_EMAILS, AUTH_SECRET, etc.
npm install
npx prisma migrate deploy   # creates the SQLite database + schema
npm run db:seed             # seeds the 5 components, sample tasks/quizzes/resources
npm run dev
```

Visit `http://localhost:3000`. The seed script prints two demo accounts:

- **Admin**: whichever email you set as the first entry in `ADMIN_EMAILS`
  (password `ChangeMe123!`)
- **Student**: `demo.student@example.com` (password `DemoPass123!`)

**Change both passwords (or delete the accounts) before using this for
real.** The seeded admin password is a placeholder.

### Registering real accounts

Anyone can register at `/register`. The very first account ever created, and
any email listed in the comma-separated `ADMIN_EMAILS` env var, is promoted
to `ADMIN` automatically; everyone else is a `STUDENT`.

### Email delivery for certificates

Certificates always generate and are downloadable from the dashboard, with
or without email configured. To also have them emailed automatically the
moment a student hits 100%, set the `SMTP_*` variables in `.env` (see
`.env.example`). If they're left blank, the app just skips sending and the
download still works.

### Editing the curriculum

Components, tasks, points, and quiz questions live in `prisma/seed.ts`. Edit
that file and re-run `npm run db:seed` (it's idempotent — safe to re-run) to
update the curriculum without touching application code.

## Project structure

```
prisma/schema.prisma       Data model (users, components, tasks, quizzes, progress, updates, resources, certificates)
prisma/seed.ts             Curriculum content + demo accounts
src/lib/auth.ts            NextAuth config (credentials provider)
src/lib/progress.ts        Per-user completion % computation (single source of truth)
src/lib/certificate.ts     PDF certificate generation (pdf-lib)
src/lib/mailer.ts          Optional SMTP email delivery
src/app/dashboard/         Progress tracking page
src/app/updates/           Updates/announcements page
src/app/resources/         Resources page
src/app/admin/             Admin roster + per-student detail
src/app/api/               Route handlers (auth, register, progress, quizzes, certificate, updates, resources)
```

## Notes on production deployment

- Swap SQLite for a hosted Postgres/MySQL database by changing the
  `datasource` provider in `prisma/schema.prisma` and `DATABASE_URL`.
- Set a strong, random `AUTH_SECRET` and a real `NEXTAUTH_URL`.
- `npm audit` flags a known advisory in `nodemailer`'s `raw` message option
  (SSRF/file-read). This app never uses that option — mail is sent with
  structured `to`/`subject`/`text`/`attachments` fields only — but keep an
  eye on upstream fixes.
