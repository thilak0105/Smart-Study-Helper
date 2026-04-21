# Smart Study Helper

Smart Study Helper is a Django-based learning assistant designed to help students upload study material (PDF), generate summaries and notes, practice with questions, track streaks, and improve day-to-day study consistency.

The project was originally built in a hackathon context and is now structured for continued development and deployment.

## 1. Core Capabilities

- User authentication with custom user model
- PDF upload and study material management
- Automatic text extraction from uploaded PDFs
- AI-oriented summarization flow with fallback summarizer
- Notes generation from summaries
- Question generation from notes (spaCy-driven with safe fallback)
- Basic streak tracking API endpoint
- Text-to-speech controls for summary playback
- PDF translation flow (currently configured for Tamil target language in route usage)

## 2. Tech Stack

- Python 3.12
- Django 5.1.x
- SQLite (default local development database)
- Optional PostgreSQL support via environment variables
- PyPDF2 for PDF text extraction
- Transformers + T5 model for advanced summarization (optional at runtime)
- spaCy for question generation (optional at runtime)
- pyttsx3 for text-to-speech
- googletrans for translation

## 3. Project Structure

```text
studyhelper/
├── manage.py
├── README.md
├── db.sqlite3                  # local runtime database (do not commit in production workflows)
├── media/                      # uploaded files (runtime data)
├── study/
│   ├── admin.py
│   ├── apps.py
│   ├── forms.py
│   ├── models.py
│   ├── urls.py
│   ├── views.py
│   ├── migrations/
│   ├── templates/
│   └── static/
└── studyhelper/
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

## 4. Application Architecture

### Main App

- App name: `study`
- Project settings package: `studyhelper`
- Entry point: `manage.py`

### Data Model Highlights

Defined in `study/models.py`:

- `CustomUser` (extends `AbstractUser`)
- `Profile` (study goal + available daily time)
- `StudyMaterial` (uploaded file metadata and summary fields)
- `Module` (generated chunks/sections of material)
- `Evaluation` (stored question-answer style evaluation fields)
- `Quiz`, `Question`, `Option`, `QuizAttempt`
- `UserProgress`, `Notification`, `Streak`

### Request Flow (High-Level)

1. User signs up/login.
2. User uploads PDF study material.
3. System extracts text from PDF.
4. Summarization attempts T5 model load and generation.
5. If model dependencies are unavailable, fallback extractive summarizer is used.
6. Summary is stored and rendered.
7. Notes/questions can be generated from summary text.

## 5. URL Endpoints

Configured in `study/urls.py`:

- `/` -> dashboard/dummy page
- `/home/` -> home page
- `/login/` -> login
- `/signup/` -> signup
- `/profile/` -> profile page
- `/upload_study_material/` -> upload route
- `/process_study_material/<id>/` -> summarization pipeline
- `/lessons/<id>/` -> lessons view for a material
- `/generate_notes/<id>/` -> notes from summary
- `/questions_form/` -> question generation UI
- `/submit_answers/` -> answer submission UI
- `/streaks/` -> streak JSON endpoint
- `/start_text_to_speech/` -> start TTS
- `/stop_text_to_speech/` -> stop TTS
- `/translate_pdf/` -> translation workflow

## 6. Database Configuration

In `studyhelper/settings.py`:

- SQLite is the default for local development.
- PostgreSQL can be enabled by setting:

```bash
USE_POSTGRES=true
POSTGRES_DB=study
POSTGRES_USER=<your_user>
POSTGRES_PASSWORD=<your_password>
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

## 7. Local Setup

### Prerequisites

- Python 3.12+
- pip
- Optional: virtual environment

### Create and Activate Environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Install Dependencies

```bash
pip install django PyPDF2 pyttsx3 googletrans==4.0.0-rc1 nest_asyncio
```

Optional advanced NLP/AI dependencies:

```bash
pip install transformers torch spacy
python -m spacy download en_core_web_sm
```

### Run Migrations

```bash
python manage.py migrate
```

### Run Development Server

```bash
python manage.py runserver 127.0.0.1:8000
```

Open: http://127.0.0.1:8000

## 8. Known Operational Notes

- If Torch is not available or broken, summary generation falls back to a pure-Python extractive summarizer.
- If spaCy model is not available, question generation falls back to a lightweight heuristic question generator.
- Uploaded files are stored under `media/` and should usually be excluded from git.
- `db.sqlite3` is local runtime state and should not be committed for team workflows.

## 9. Development Recommendations

- Move `SECRET_KEY` to environment variables before production deployment.
- Set `DEBUG=False` and configure `ALLOWED_HOSTS` for production.
- Add a formal dependency lock file (`requirements.txt` or `pyproject.toml`).
- Add test coverage for critical flows:
  - upload + summarize
  - auth
  - question generation
  - translation

## 10. Troubleshooting

### Error: missing torch dylib / transformers import failure

Use fallback behavior (already supported), or reinstall torch cleanly in your environment.

### Error: PostgreSQL authentication failed

Either:
- use SQLite default by not setting `USE_POSTGRES`, or
- correct Postgres credentials in environment variables.

### Error: missing module (googletrans / pyttsx3 / nest_asyncio)

Install missing packages:

```bash
pip install googletrans==4.0.0-rc1 pyttsx3 nest_asyncio
```

## 11. Future Enhancements

- Replace duplicated imports and repeated helper definitions in views with service-layer modules
- Add API schema and separate frontend template responsibilities
- Add async task queue for long PDF processing jobs
- Add role-based access and progress analytics dashboard
- Add CI for linting, tests, and migration checks

## 12. License

No license file is currently included. Add a LICENSE file (MIT/Apache-2.0/etc.) before public distribution.
