# TAMVA

TAMVA is financial identity and trust infrastructure. This repository currently
delivers the **frontend surfaces** of the system — the admin/institution web
portal and the customer mobile/web application — built against a Django
backend that exists as structural scaffolding, not yet a working API surface.

## Where the work stands

The frontend is the completed part of this system. The backend is a
**template**: it defines the intended architecture, folder structure, and
domain boundaries, but its endpoints, business logic, and data models are not
yet implemented. Treat `apps/backend/` as a placeholder to configure and build
into, not as a working service.

| Application surface | Location | Status | Notes |
| --- | --- | --- | --- |
| Admin/institution/operations web | `apps/admin/` | **Built** | React + TypeScript portal for staff and institutional users |
| Customer application | `apps/mobile/` | **Built** | Expo/React Native app covering Android, iOS, and web |
| Django backend | `apps/backend/` | **Scaffold only** | Folder structure and domain boundaries are defined; APIs and business logic are not yet implemented |

Both frontend apps currently run against local mock fixtures and a local
Zustand store rather than live backend data, since the backend has no
authoritative endpoints yet.

## Architecture (intended)

The long-term design is a modular monolith: one Django application with
domain-owned modules (identity, consent, risk, ledger, passport,
case-management), PostgreSQL as the source of truth, and Redis for caching and
Celery. Institutions provide the shared-database tenancy boundary. None of
this is live yet — it's the target the backend scaffold is structured around.

```text
                    PostgreSQL
                 SOURCE OF TRUTH
                    (planned)
                        ↑
                        │
                  Django Backend
              TEMPLATE / SCAFFOLD
                        │
                 REST / OpenAPI
                    (planned)
                 ┌──────┴──────┐
                 │             │
                 ↓             ↓
        Admin/Operations   Customer App
           React (built)   React Native
                          (built)
```

Once the backend is implemented, both frontend apps are expected to consume
it through versioned REST APIs (`/api/v1/`) rather than mock fixtures. See
[system overview](docs/architecture/system-overview.md),
[tenancy model](docs/architecture/tenancy.md), and
[architecture decisions](docs/adr/) for the full intended design.

## Frontend applications

### Admin, institutional, and operations web (`apps/admin/`)

Built and functional. Serves TAMVA platform staff, institution administrators,
risk analysts, investigators, operations staff, security and compliance
staff, auditors, and API/integration developers. Not customer-facing. One
application with backend-controlled menus, roles, permissions, and
institution scopes — not separate frontends per role, though menu/role
control is currently stubbed pending real backend authorization.

- React 19 and TypeScript
- Vite
- TanStack Router, Query, and Table
- Tailwind CSS and owned shadcn-style components
- Lucide icons

### Customer application (`apps/mobile/`)

Built and functional as a shared codebase for Android, iOS, and web — there
is no separate customer web app. Navigation adapts from bottom tabs on phones
to a sidebar on tablets/desktop. The current release includes Home, Activity,
Profile, Passport, Protection, Confidence, Consent, Notifications, and
Passport creation flows, all running on local mock fixtures until the
backend domain APIs exist.

- React Native and Expo
- TypeScript and Expo Router
- Android, iOS, and web targets
- TanStack Query
- NativeWind
- React Hook Form and Zod
- Expo SecureStore

## Backend (`apps/backend/`) — scaffold, not implementation

The backend folder structure, domain boundaries (`apps/backend/domains/`),
and stable primitives (`apps/backend/packages/`) are defined, along with
Docker/Compose wiring, migrations tooling, and CI hooks. This is a template
for the team to build into — authentication, tenancy, consent, risk, ledger,
profile, passport, and case-decision logic are not implemented yet. Each
directory under `apps/backend/domains/` has a README stating that domain's
intended responsibility.

Planned stack (not yet exercised by real endpoints):

- Python 3.13, Django 5.2, Django REST Framework, Celery
- PostgreSQL 17, Redis 7
- REST with OpenAPI/Swagger via drf-spectacular

## Technology stack summary

### Testing and quality

- pytest and pytest-django (backend, once implemented)
- Ruff linting and formatting
- mypy with Django type support
- coverage, pre-commit
- ESLint for admin and mobile clients

### CI/CD

- GitHub Actions for pull requests and changes to `main` or `tamva`

## Repository structure

```text
tamva/
├── apps/
│   ├── backend/          Django scaffold — structure defined, logic pending
│   ├── admin/            Institutional React/Vite application (built)
│   └── mobile/           Universal Expo customer application (built)
├── packages/contracts/   Shared TypeScript schemas and OpenAPI output (placeholder until backend ships)
├── docs/                 Architecture, ADRs, API, security, and runbooks
├── scripts/              Container entry points and operational helpers
├── pnpm-workspace.yaml   Workspace package ownership
├── .github/              CI workflow and contribution templates
├── Makefile
├── docker-compose.yml
├── README.md
├── CONTRIBUTING.md
└── SECURITY.md
```

## Prerequisites

- Git with access to `Ja-moah/tamva`
- Node.js 22 or newer and pnpm 11 or newer (via Corepack) — required for frontend work
- Docker Engine and Docker Compose v2 — only needed if standing up the backend scaffold
- GNU Make

Python 3.13+ is needed on the host only when running backend tooling outside Docker.

## Getting started (frontend-focused)

```bash
git clone git@github.com:Ja-moah/tamva.git
cd tamva
git checkout tamva

make frontend-install
```

### Admin portal

```bash
make admin-dev
```

Runs the admin portal with Vite HMR on <http://localhost:3000/>.

### Customer app

```bash
cp apps/mobile/.env.example apps/mobile/.env
make mobile-start
```

Use `make mobile-web` for the browser target. Expo prints the Metro
development URL and QR code at runtime. Configure
`EXPO_PUBLIC_API_BASE_URL` per target:

| Runtime | Django API base URL |
| --- | --- |
| iOS simulator | `http://localhost:8000` |
| Android emulator | `http://10.0.2.2:8000` |
| Local web browser | `http://localhost:8000` |
| Physical device | `http://<development-machine-LAN-address>:8000` |

Note: until the backend is implemented, these apps run on mock fixtures
regardless of the configured API base URL.

### Backend scaffold (optional, for backend work)

```bash
cp .env.example .env
make bootstrap
```

Brings up Django, PostgreSQL, Redis, and Celery per the scaffold. There are
no authoritative endpoints beyond `/health/` yet.

## Developer commands (frontend-relevant)

| Command | Purpose |
| --- | --- |
| `make frontend-install` | Install locked web and mobile dependencies |
| `make frontend-dev` | Run the admin portal with Vite HMR |
| `make frontend-build` | Build the admin web application for production |
| `make frontend-test` | Run the admin web application test suite |
| `make frontend-typecheck` | Type-check the admin web application and mobile |
| `make admin-dev` | Run the admin portal with Vite HMR on port `3000` |
| `make admin-build` | Build the admin portal for production |
| `make admin-test` | Run admin portal tests |
| `make mobile-start` | Start the Expo customer app |
| `make mobile-web` | Start the Expo customer app for web |
| `make mobile-android` | Open the Expo Android workflow |
| `make mobile-ios` | Open the Expo iOS workflow |
| `make mobile-lint` | Lint shared Android, iOS, and web customer code |
| `make mobile-build` | Export Android, iOS, and web customer bundles |
| `make clients-check` | Verify both client applications |

Run `make help` for the full command list, including backend-scaffold commands.

## Team development workflow

```text
main
  ↑
tamva
  ↑
feature branches
```

- `main` is the stable, accepted release branch.
- `tamva` is the shared integration and acceptance branch.
- Feature branches start from the latest `tamva`.
- Nobody develops or pushes feature commits directly on `main` or `tamva`.
- Feature pull requests target `tamva`.
- Once integrated work is fully tested and accepted, a release pull request promotes `tamva` to `main`.
- This repository does not use a `develop` branch.

### Start a feature

```bash
git checkout tamva
git pull origin tamva
git checkout -b feat/<feature-name>
```

Use a focused name such as `feat/admin-risk-view` or `feat/mobile-passport-flow`.
Work inside the owning app, avoid unrelated refactors, and include tests and
documentation.

### Submit work for review

```bash
make check
make test

git add .
git commit -m "feat(domain): short description"
git push -u origin feat/<feature-name>
```

Open a pull request from `feat/<feature-name>` into `tamva`. Complete the
pull-request template and obtain review before merge.

Read [CONTRIBUTING.md](CONTRIBUTING.md) for the review and change-management rules.

## Before your AI touches the code

Before allowing an AI agent to inspect or modify the repository, instruct it to read:

- `AGENTS.md`
- `AI_GOVERNANCE.md`
- the assigned domain skill
- relevant ADRs under `docs/adr/`
- the relevant module README under `apps/<domain>/README.md`

`AGENTS.md` and `AI_GOVERNANCE.md` are currently **pending** and do not yet
exist in this repository. Do not tell an AI agent that it has read them until
the team creates and reviews them. Domain skills are also team-assigned
resources and are not fabricated by this setup update.
