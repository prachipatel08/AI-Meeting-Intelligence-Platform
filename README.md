# AI Meeting Intelligence Platform

**Tagline:** Turn Every Conversation Into Action.

An AI-powered web application that transforms uploaded audio/video meetings into timestamped transcripts and structured meeting intelligence: summaries, topics, decisions, action items, owners, deadlines, questions, risks, unresolved issues, and follow-ups.

## Technology stack

- **Frontend:** React, Vite, Tailwind CSS, Axios, React Router, Framer Motion
- **Backend:** Python, Django, Django REST Framework
- **Authentication:** JWT (`djangorestframework-simplejwt`)
- **Application data:** MongoDB Atlas accessed with PyMongo
- **AI:** Groq API (speech transcription and LLM analysis)
- **Background processing:** Celery + Redis
- **Media:** FFmpeg
- **Real-time status (planned):** Django Channels/WebSockets
- **Exports (planned):** PDF, DOCX, TXT, JSON, SRT, VTT

## Important database decision

Django's standard ORM does **not** natively support MongoDB as a normal first-party database backend. To avoid a fragile setup, this starter uses:

- Django's default SQLite database for Django users, groups, permissions, migrations, and admin during development.
- MongoDB Atlas + **PyMongo** for application documents such as meetings, transcript segments, speaker labels, analysis results, and usage records.

Do not try to set `DATABASES['default']['ENGINE']` to a normal Django ORM MongoDB backend unless the team deliberately selects, installs, and tests a compatible solution. The MongoDB integration must be implemented in the project code; installing PyMongo alone does not automatically make Django models save to MongoDB.

## Team setup overview

Each team member installs Git, Node.js LTS, Python, and FFmpeg. Docker is optional for local development but useful later. The team lead creates one MongoDB Atlas development database and shares the connection details securely with authorized teammates—not in GitHub, screenshots, or chat channels.

### Repository layout

```text
ai-meeting-intelligence-platform/
├── frontend/                 # React + Vite
├── backend/
│   ├── config/               # Django project settings/URLs/ASGI/WSGI
│   ├── apps/                 # Django API apps
│   ├── media_processing/     # FFmpeg and media helpers
│   ├── storage/              # private file storage helpers
│   ├── tests/
│   ├── manage.py
│   └── requirements.txt
├── docs/
│   ├── setup-guide.md
│   └── api-contracts.md
├── infra/
├── scripts/
├── .env.example
├── .gitignore
└── README.md
```

## Prerequisites

- Git
- Node.js LTS (includes npm)
- Python 3.11 or 3.12 (use the version agreed by the team)
- FFmpeg installed and available on PATH
- MongoDB Atlas account and shared development cluster
- Groq API key for backend AI calls
- Redis server or a hosted Redis-compatible service for Celery
- Optional: Docker Desktop

Verify installations in PowerShell:

```powershell
git --version
node --version
npm --version
py --version
ffmpeg -version
```

If `py` is unavailable, try `python --version`.

## First-time setup (Windows PowerShell)

### 1. Clone the repository

```powershell
git clone https://github.com/YOUR-ORG-OR-USERNAME/ai-meeting-intelligence-platform.git
cd ai-meeting-intelligence-platform
```

### 2. Create your local environment file

From the repository root:

```powershell
Copy-Item .env.example .env
```

Fill in the values in `.env` using credentials supplied through the team's approved private channel. **Never commit `.env`.** Keep `.env.example` free of real credentials.

### 3. Set up the backend

```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
cd ..
```

If PowerShell blocks activation, use this for the current terminal only, if permitted by your device policy:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

### 4. Set up the frontend

```powershell
cd frontend
npm install
cd ..
```

### 5. Run Django checks

After the team has implemented settings and the database connection code:

```powershell
cd backend
python manage.py check
python manage.py migrate
python manage.py runserver
```

The API usually runs at `http://127.0.0.1:8000/`. The SQLite migrations are for Django's own auth/admin tables in this starter approach. MongoDB collections are accessed by application code using PyMongo.

### 6. Run the frontend

Open a second terminal at the repository root:

```powershell
cd frontend
npm run dev
```

Vite usually prints `http://localhost:5173/`.

### 7. Run background processing (when configured)

Redis must be running and `REDIS_URL` must point to it. In one terminal:

```powershell
cd backend
.\.venv\Scripts\Activate.ps1
celery -A config worker --loglevel=info --pool=solo
```

The `-A config` value assumes the Celery app is configured in `backend/config/`. This command will not work until the team adds the Celery application module and task configuration.

## MongoDB Atlas setup for the team

1. Open https://www.mongodb.com/cloud/atlas and create/sign in to an Atlas account.
2. Create a **shared development project** for this application and create a cluster (choose a free/shared tier if available in your region).
3. In **Database Access**, create a dedicated database user for the application. Use a strong unique password. Do not use your personal Atlas account password.
4. In **Network Access**, add each teammate's current public IP address. For a short-lived class demo only, Atlas may allow `0.0.0.0/0`, but it exposes the cluster endpoint to connection attempts from anywhere and is not recommended. Use least-privilege network access wherever possible.
5. Select **Connect → Drivers → Python** and copy the URI template.
6. Replace the password placeholder with the database user's password, URL-encoding special characters if necessary. Keep the URI private.
7. Agree on one database name, for example `ai_meeting_intelligence_dev`. Put the same URI/database name in each member's local `.env` file.
8. Test the connection with the project’s backend connection-check command once it has been implemented. PyMongo can connect, but the project still needs code that reads `MONGODB_URI` and accesses collections.
9. Use a development database only. Never upload confidential meeting recordings or sensitive production data to a shared student/demo cluster without authorization and a clear retention policy.

**All members using the same URI/database will share the same data.** This is useful for collaborative development, but anyone with write access can change or delete shared records. Use separate collections or a separate database for tests; never run destructive tests against shared development data.

## Environment variables

See `.env.example`. Variables should be read by backend code via `python-dotenv` or Django settings. The frontend must only receive public configuration such as `VITE_API_BASE_URL`; never put Groq, MongoDB, Redis, or storage secrets in `VITE_*` variables.

## Git workflow

Do not push directly to `main`. Create a feature branch, commit only your feature, push it, and open a Pull Request.

```powershell
git switch main
git pull origin main
git switch -c feature/short-feature-name
git status
git add .
git commit -m "feat: describe your change"
git push -u origin feature/short-feature-name
```

Then open a Pull Request on GitHub and request a review.

## Security checklist

- Never commit `.env`, API keys, passwords, tokens, uploaded recordings, or generated private reports.
- Use `.gitignore` and inspect `git status` before every commit.
- Keep Groq calls on the backend; never call Groq directly from React.
- Restrict meeting and transcript access to the owning user and authorized managers/admins.
- Keep uploaded files in private storage with randomized names; do not use public URLs by default.
- Add file type/size validation, rate limits, logging, retention/deletion policies, and retries.
- Speaker labels are AI-inferred conversational turns, not guaranteed biometric voice identification; allow manual corrections.
- Rotate any secret that was accidentally committed.

## Current implementation status

This README describes the intended project and setup. It does not mean all features, settings, MongoDB connection helpers, Celery tasks, endpoints, or export handlers have already been implemented. Follow the team’s API contracts and implementation checklist.

## Troubleshooting

- **`MONGODB_URI` is empty:** check the local `.env` file and confirm the backend loads it.
- **MongoDB authentication fails:** verify the database user, password URL encoding, database name, and Atlas Network Access IP list.
- **MongoDB timeout:** check internet access, cluster status, and Network Access rules.
- **`ffmpeg` not recognized:** install FFmpeg and add its executable folder to PATH, then reopen the terminal.
- **Groq authentication fails:** verify the backend-only `GROQ_API_KEY` and account/API access.
- **Celery cannot connect:** start Redis and verify `REDIS_URL`.
- **CORS error:** confirm the frontend URL is allowed by Django CORS settings.
- **`manage.py` missing:** run Django commands from the `backend/` directory.
