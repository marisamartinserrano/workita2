# Workita — Your AI Job Companion

**Workita** is an AI-powered job search companion that helps candidates navigate every stage of the job application process — from profile setup and CV analysis to candidature tracking and interview preparation.

---

## ✨ Features

### User Journeys

| Journey | Description |
|---|---|
| **Registration / Login** | Secure sign-in via Google SSO (OAuth 2.0) |
| **Onboarding** | Build your candidate profile, upload your CV, and set job preferences |
| **New Candidature** | Paste a job description and get an AI-powered match report |
| **Candidatures Hub** | Track all your applications and their selection process stages |
| **Selection Process** | Get AI-generated interview prep questions for each stage |
| **CV Analysis** | AI parses your CV and simulates how an ATS system reads it |
| **LinkedIn Analysis** | Get role-targeted LinkedIn profile recommendations |
| **Glossary** | Extract and define key terms from any job description |
| **Quizzes** | Test your knowledge with AI-generated multiple-choice quizzes |
| **Closing** | Manage offer decisions and wrap up applications |

### AI Capabilities (Google Genkit + Gemini 2.5 Flash)

- **Job Analysis** — match score, strengths, gaps, ATS keywords, CV & LinkedIn recommendations, networking guidance
- **Position Summary** — concise structured summary extracted from any job posting
- **CV Analysis** — skills, experience, education, gap detection, full ATS simulation with scoring
- **LinkedIn Analysis** — suitability assessment against target role, harmonization tips, profile recommendations
- **Interview Prep** — stage-specific questions with tips and sample answers grounded in the job posting
- **Glossary Generation** — industry and technical term definitions from a job description
- **Quiz Generation** — difficulty-tiered multiple-choice questions on any topic for a given role
- **Job Metadata Extraction** — auto-fill job title, company, role, seniority, location, and work mode from a posting

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, TypeScript, Tailwind CSS, Vite |
| Backend | Node.js, Express |
| Database | PostgreSQL 16, Drizzle ORM |
| AI Framework | Google Genkit + Google Gemini 2.5 Flash |
| Authentication | Google SSO (OAuth 2.0), Passport.js |
| Infrastructure | Docker Compose, nginx |

---

## 🚀 Getting Started

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- A **Google OAuth 2.0** client (Client ID + Secret) — [Google Cloud Console](https://console.cloud.google.com/)
- A **Google Gemini API key** — [Google AI Studio](https://aistudio.google.com/)

### 1. Clone the repo

```bash
git clone https://github.com/marisamartinserrano/workita2.git
cd workita2
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` and fill in your credentials:

```env
# Google OAuth 2.0
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# Google Gemini / Genkit
GOOGLE_GENAI_API_KEY=your_google_genai_api_key

# Auth secrets (generate long random strings)
JWT_SECRET=your_long_random_jwt_secret
SESSION_SECRET=your_long_random_session_secret

# App
FRONTEND_URL=http://localhost:8080
```

> The database variables (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`) default to `workita` — no changes needed for local development.

### 3. Start the app

```bash
docker compose up --build
```

The app will be available at **http://localhost:8080**.

---

## 🧑‍💻 Development Mode

Run the frontend and backend locally without Docker:

```bash
npm install
npm run dev
```

- Frontend: http://localhost:5173
- Backend: http://localhost:3001

You will need a local PostgreSQL instance (or keep the Docker DB running on port 5432).

### Database commands

```bash
npm run db:generate   # Generate Drizzle migrations from schema changes
npm run db:migrate    # Run pending migrations
npm run db:push       # Push schema directly to the database (dev only)
npm run db:studio     # Open Drizzle Studio (database GUI)
```

---

## 📁 Project Structure

```
workita2/
├── src/                      # React frontend
│   ├── components/           # Shared UI components (Layout, ProtectedRoute)
│   ├── contexts/             # React contexts (AuthContext)
│   ├── lib/                  # API client utilities
│   └── pages/                # Page components (one per journey)
├── server/                   # Express backend
│   ├── ai/
│   │   ├── genkit.ts         # Genkit / Gemini initialisation
│   │   └── flows.ts          # All AI flows (analyzeJob, cvAnalysis, etc.)
│   ├── db/
│   │   ├── schema.ts         # Drizzle schema (users, profiles, candidatures, stages)
│   │   ├── index.ts          # DB connection
│   │   └── migrate.ts        # Migration runner
│   ├── middleware/
│   │   └── auth.ts           # JWT auth middleware
│   ├── routes/
│   │   ├── auth.ts           # Google OAuth routes
│   │   ├── profile.ts        # Profile CRUD + CV/LinkedIn upload & analysis
│   │   ├── candidatures.ts   # Candidature CRUD + stage management
│   │   └── ai.ts             # AI endpoint routes (interview prep, glossary, quiz)
│   ├── utils/
│   │   └── stripHtml.ts      # HTML stripping utility
│   └── index.ts              # Express app entry point
├── drizzle/
│   └── migrations/           # Auto-generated SQL migration files
├── openspec/                 # OpenSpec design workflow
│   ├── specs/                # Feature specifications
│   └── changes/              # Change proposals and archives
├── docker-compose.yml        # Production Docker Compose (3 services: db, server, client)
├── docker-compose.dev.yml    # Development Docker Compose
├── Dockerfile.server         # Server Docker image
├── Dockerfile.client         # Client Docker image (nginx)
├── nginx.conf                # nginx config (serves SPA + proxies API)
└── drizzle.config.ts         # Drizzle ORM config
```

---

## 🔑 Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → APIs & Services → Credentials
2. Create an **OAuth 2.0 Client ID** (Web application)
3. Add **Authorised redirect URIs**:
   - `http://localhost:3001/api/auth/google/callback` (local)
4. Copy the Client ID and Client Secret into `.env`

---

## 📄 License

MIT
