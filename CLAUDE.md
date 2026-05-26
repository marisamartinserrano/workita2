# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

---

## About

**Workita** is an AI-powered job search companion. It helps candidates navigate the job application process with intelligent recommendations at every stage — from CV analysis and candidature tracking to interview preparation and LinkedIn optimisation.

The app is a full-stack TypeScript monorepo: a React SPA frontend, an Express backend, PostgreSQL via Drizzle ORM, and Google Genkit (Gemini 2.5 Flash) for all AI features.

---

## Running the App

### Production (Docker Compose)

```bash
docker compose up --build
```

App available at **http://localhost:8080**. Three services: `db` (PostgreSQL), `server` (Node/Express on port 3001), `client` (nginx on port 8080).

### Development

```bash
npm run dev           # Concurrently runs frontend (Vite) + backend (tsx watch)
```

- Frontend: http://localhost:5173
- Backend API: http://localhost:3001

Requires a local PostgreSQL instance or keep the Docker DB running on port 5432.

---

## Project Structure

```
src/                        # React frontend (Vite + TypeScript)
  components/               # Layout, ProtectedRoute
  contexts/AuthContext.tsx  # Auth state (JWT + user)
  lib/api.ts                # Typed fetch wrapper for all API calls
  pages/                    # One file per page/journey (see Pages below)

server/                     # Express backend (TypeScript, compiled to dist/server)
  ai/
    genkit.ts               # Genkit initialisation (Gemini 2.5 Flash)
    flows.ts                # All AI flows (see AI Flows below)
  db/
    schema.ts               # Drizzle table definitions
    index.ts                # DB connection pool
    migrate.ts              # Migration runner (called on server startup)
  middleware/auth.ts        # requireAuth — verifies JWT, attaches req.user
  routes/
    auth.ts                 # GET /api/auth/google, /callback, /me, /logout
    profile.ts              # GET/PUT /api/profile, CV upload, LinkedIn analysis
    candidatures.ts         # CRUD /api/candidatures, stage management
    ai.ts                   # POST /api/ai/interview-prep, /glossary, /quiz, /summarize-position
  utils/stripHtml.ts        # Strip HTML tags from job description text
  index.ts                  # Express app — mounts routes, starts server

drizzle/migrations/         # Auto-generated SQL migrations (do not edit manually)
openspec/                   # OpenSpec design workflow (specs + change archives)
```

---

## Pages (src/pages/)

| File | Route | Description |
|---|---|---|
| `Login.tsx` | `/login` | Google SSO sign-in page |
| `Onboarding.tsx` | `/onboarding` | Profile setup wizard (role, seniority, CV upload) |
| `Dashboard.tsx` | `/dashboard` | Home — summary of candidatures and quick actions |
| `Profile.tsx` | `/profile` | View and edit candidate profile |
| `CvAnalysis.tsx` | `/cv-analysis` | CV upload + ATS simulation results |
| `LinkedinAnalysis.tsx` | `/linkedin-analysis` | LinkedIn profile assessment |
| `NewCandidature.tsx` | `/new-candidature` | Paste job description → AI match report |
| `Candidatures.tsx` | `/candidatures` | Candidatures hub — list and filter |
| `SelectionProcessList.tsx` | `/candidatures/:id` | Stages list for a single candidature |
| `SelectionProcess.tsx` | `/candidatures/:id/stage/:stage` | Interview prep for a specific stage |
| `Glossary.tsx` | `/glossary` | AI-generated glossary from a job description |
| `Quizzes.tsx` | `/quizzes` | AI-generated multiple-choice quizzes |
| `Closing.tsx` | `/closing` | Offer management and application closing |

---

## Database Schema (server/db/schema.ts)

### `users`
| Column | Type | Notes |
|---|---|---|
| id | text PK | UUID |
| googleId | text UNIQUE | |
| email | text UNIQUE | |
| name | text | |
| picture | text | |
| createdAt | timestamp | |

### `profiles`
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| userId | text FK → users | |
| role, seniority, industry, location, workMode, salaryExpectations, preferredCompanies | text | Job preferences |
| cvText | text | Extracted plain text from uploaded CV |
| cvFileName, cvFileData, cvFileType | text | Stored CV file (base64) |
| cvAnalysisResult | jsonb | Result of `cvAnalysisFlow` |
| linkedinUrl | text | |
| linkedinAnalysisResult | jsonb | Result of `linkedinAnalysisFlow` |

### `candidatures`
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| userId | text FK → users | |
| jobTitle, company | text | |
| jobUrl, jobDescription, positionSummary | text | Raw posting data |
| status | varchar(50) | `applied` \| `in_progress` \| `closed` |
| matchPercentage | integer | 0–100 from analyzeJobFlow |
| analysis | jsonb | Full result of `analyzeJobFlow` |
| role, seniority, location, workMode, industry, labels | text/jsonb | Enriched metadata |

### `candidature_stages`
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| candidatureId | text FK → candidatures | |
| stage | text | e.g. "HR Screening", "Technical Interview" |
| status | varchar(50) | `pending` \| `scheduled` \| `completed` |
| notes | text | |
| scheduledAt, completedAt | timestamp | |

---

## AI Flows (server/ai/flows.ts)

All flows use Google Genkit with Gemini 2.5 Flash and return typed, Zod-validated output.

| Flow | Trigger | Output |
|---|---|---|
| `analyzeJobFlow` | New candidature creation | Match %, strengths, gaps, ATS keywords, CV/LinkedIn recs, networking tips |
| `summarizePositionFlow` | After job description paste | 6-line structured position summary |
| `extractJobMetadataFlow` | New candidature quick-add | jobTitle, company, role, seniority, location, workMode, industry |
| `cvAnalysisFlow` | CV upload in profile | Skills, experience, education, gaps, full ATS simulation with score |
| `linkedinAnalysisFlow` | LinkedIn section in profile | Suitability strengths/gaps, harmonization tips, profile recommendations |
| `linkedinRecommendationsFlow` | (deprecated — use linkedinAnalysisFlow) | Generic LinkedIn improvement recs |
| `interviewPrepFlow` | Selection process stage view | 5 interview questions with tips and sample answers |
| `glossaryFlow` | Glossary page | 8–15 term definitions from a job description |
| `quizFlow` | Quizzes page | 5 multiple-choice questions with explanations |

---

## API Routes

All routes are prefixed with `/api`.

### Auth (`/api/auth`)
- `GET /google` — initiates Google OAuth redirect
- `GET /google/callback` — OAuth callback, sets JWT cookie
- `GET /me` — returns current authenticated user
- `GET /logout` — clears session

### Profile (`/api/profile`)
- `GET /` — get current user's profile
- `PUT /` — update profile fields
- `POST /upload-cv` — upload CV (PDF/DOCX), extract text, trigger cvAnalysisFlow
- `POST /analyze-cv` — re-run cvAnalysisFlow on stored CV text
- `POST /analyze-linkedin` — run linkedinAnalysisFlow and persist result

### Candidatures (`/api/candidatures`)
- `GET /` — list all candidatures for current user
- `POST /` — create new candidature (triggers analyzeJobFlow + summarizePositionFlow + extractJobMetadataFlow)
- `GET /:id` — get single candidature
- `PUT /:id` — update candidature
- `DELETE /:id` — delete candidature
- `GET /:id/stages` — list stages for a candidature
- `POST /:id/stages` — add a stage
- `PUT /:id/stages/:stageId` — update a stage

### AI (`/api/ai`)
- `POST /interview-prep` — generate interview prep for a stage
- `POST /glossary` — generate glossary from a job description
- `POST /quiz` — generate a quiz
- `POST /summarize-position` — summarize a job posting

---

## Key Conventions

- **Auth**: All protected routes use `requireAuth` middleware (JWT from `Authorization: Bearer` header or cookie). Frontend stores JWT in localStorage and sends it with every request via `src/lib/api.ts`.
- **CV upload**: Accepts PDF and DOCX. Text is extracted server-side (`pdf-parse` for PDF, `mammoth` for DOCX) and stored as plain text in `profiles.cvText`.
- **AI prompts**: All prompts request `ONLY valid JSON` output. Responses are parsed with `JSON.parse`. Use the `extractJson()` helper for flows that sometimes return fenced code blocks.
- **Migrations**: Run automatically on server startup via `server/db/migrate.ts`. To make schema changes: edit `server/db/schema.ts` → run `npm run db:generate` → commit the new migration file.
- **TypeScript**: Strict mode on. Server code compiles to `dist/server/` via `tsconfig.json` + `server/tsconfig.json`. Frontend builds to `dist/` via Vite.
- **Frontend routing**: React Router v7, all routes are client-side, nginx serves `index.html` for all non-`/api` paths.

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `GOOGLE_CLIENT_ID` | ✅ | Google OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | ✅ | Google OAuth client secret |
| `GOOGLE_GENAI_API_KEY` | ✅ | Google Gemini API key (via AI Studio) |
| `JWT_SECRET` | ✅ | Secret for signing JWTs |
| `SESSION_SECRET` | ✅ | Secret for express-session |
| `DATABASE_URL` | ✅ | PostgreSQL connection string |
| `POSTGRES_USER` | Docker | DB username (default: `workita`) |
| `POSTGRES_PASSWORD` | Docker | DB password (default: `workita`) |
| `POSTGRES_DB` | Docker | DB name (default: `workita`) |
| `FRONTEND_URL` | ✅ | Allowed CORS origin (default: `http://localhost:8080`) |
| `PORT` | — | Server port (default: `3001`) |

---

## OpenSpec Workflow

This repo uses an **OpenSpec** design workflow to manage feature changes. Specs live in `openspec/specs/` and completed changes are archived in `openspec/changes/archive/`.

Available skills:
- `/openspec-propose` — propose a new feature change
- `/openspec-apply-change` — implement tasks from a pending change
- `/openspec-archive-change` — archive a completed change
- `/openspec-explore` — explore ideas before writing a spec

---

## npm Scripts

```bash
npm run dev            # Start frontend + backend in watch mode (concurrently)
npm run dev:client     # Vite dev server only
npm run dev:server     # tsx watch server/index.ts only
npm run build          # TypeScript compile + Vite production build
npm run lint           # ESLint
npm run db:generate    # Generate Drizzle migration from schema diff
npm run db:migrate     # Run pending migrations
npm run db:push        # Push schema to DB without migration files (dev only)
npm run db:studio      # Open Drizzle Studio
```
