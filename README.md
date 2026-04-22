<div align="center">

# Smart Study Helper

**An AI-powered learning platform built for cognitive accessibility — helping students with ADHD, dyslexia, and learning disabilities study smarter through intelligent summarization, adaptive quizzes, multilingual support, and progress analytics.**

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Django](https://img.shields.io/badge/Django-5.1-092E20?style=flat-square&logo=django&logoColor=white)](https://djangoproject.com)
[![HuggingFace](https://img.shields.io/badge/HuggingFace-Transformers-FFD21F?style=flat-square&logo=huggingface&logoColor=black)](https://huggingface.co)
[![spaCy](https://img.shields.io/badge/spaCy-NLP-09A3D5?style=flat-square&logo=spacy&logoColor=white)](https://spacy.io)

[Overview](#overview) · [Features](#features) · [Architecture](#architecture) · [Tech Stack](#tech-stack) · [Installation](#installation) · [Usage](#usage) · [API Reference](#api-reference) · [Configuration](#configuration) · [Roadmap](#roadmap) · [Contributors](#contributors)

</div>

---

## Overview

Smart Study Helper addresses a genuine gap in educational tooling: most learning platforms are built for neurotypical users, leaving students with ADHD, dyslexia, and other cognitive differences without effective study support.

This platform processes uploaded PDF study materials through a multi-stage NLP pipeline — extracting text, generating structured T5-based summaries, breaking content into manageable modules, and producing comprehension quizzes via spaCy NER analysis. Accessibility features including text-to-speech playback and multilingual translation into 7 Indian languages are built in as first-class capabilities, not afterthoughts.

The system is designed with graceful degradation in mind: when heavy ML dependencies (PyTorch, Transformers) are unavailable, it falls back to a pure-Python extractive summarizer and heuristic question generator, ensuring the application remains functional in resource-constrained environments.

---

## Features

### PDF Processing & Summarization
- Upload any study material PDF via the dashboard
- PyPDF2 extracts text page-by-page with encoding normalization
- T5-base transformer generates abstractive summaries using beam search (`num_beams=4`, `max_length=500`, `min_length=150`)
- Summaries are cleaned, deduplicated, and rendered in both paragraph and bullet-point formats
- Language is simplified for reduced cognitive load — shorter sentences, plainer vocabulary
- Fallback: pure-Python extractive summarizer when Torch is unavailable

### Module-Based Learning
- PDFs are automatically divided into 10-page modules for paced consumption
- Each module carries a title, sequence number, page range, and estimated completion time (15 min/page)
- Students progress sequentially through modules at their own pace

### Adaptive Quiz Generation
- spaCy `en_core_web_sm` generates comprehension questions from study notes using NER, noun chunk extraction, and verb ROOT analysis
- Question types: named entity questions (`"What is X?"`), verb-based questions (`"How does X affect Y?"`), and concept questions
- Answers submitted in-platform with immediate scoring and feedback
- Fallback: lightweight heuristic question generator when spaCy is unavailable

### Text-to-Speech
- pyttsx3 reads summaries aloud at a configurable speech rate (default: 150 wpm)
- Start/stop controls exposed via REST API endpoints
- Designed for auditory learners and students with reading difficulties

### Multilingual Translation
- Translates extracted PDF text into 7 Indian regional languages: Tamil, Malayalam, Telugu, Kannada, Hindi, Gujarati, Bengali
- Chunked processing (15,000 chars/chunk) handles long documents without truncation

### Progress Tracking & Streaks
- Per-user tracking of completed modules, current module position, and rolling average quiz scores
- Daily streak system with current streak and longest streak counters
- In-app notifications for study reminders and milestone achievements

### Authentication & User Management
- Custom Django `AbstractUser` extension with per-user study goals and daily availability settings
- Full signup, login, logout flow with CSRF protection

---

## Architecture

### Processing Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                        User Uploads PDF                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
             ┌────────────────┐
             │  PyPDF2 Text   │
             │  Extraction    │
             └───────┬────────┘
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
  ┌───────────────┐     ┌───────────────────────────────────┐
  │Module Splitter│     │       T5 Summarizer (t5-base)      │
  │10 pages/module│     │  beam search | max_length=500      │
  │15 min/page est│     │  ↓ fallback: extractive summarizer │
  └───────────────┘     └──────────────┬────────────────────┘
                                       │
                        ┌──────────────┼──────────────┐
                        │              │               │
                        ▼              ▼               ▼
               ┌──────────────┐  ┌─────────┐  ┌─────────────────┐
               │  pyttsx3 TTS │  │googletrs│  │  spaCy Quiz Gen │
               │  150 wpm     │  │7 langs  │  │  NER + chunks   │
               └──────────────┘  └─────────┘  └────────┬────────┘
                                                        │
                                                        ▼
                                              ┌──────────────────┐
                                              │  User Submits    │
                                              │  Answers         │
                                              └────────┬─────────┘
                                                       │
                                                       ▼
                                              ┌──────────────────┐
                                              │ Progress Update  │
                                              │ + Streak Refresh │
                                              └──────────────────┘
```

### Project Structure

```
smart-study-helper/
│
├── manage.py
├── requirements.txt
├── README.md
│
├── studyhelper/                    # Django project configuration
│   ├── settings.py                 # Database, installed apps, media config
│   ├── urls.py                     # Root URL dispatcher
│   ├── asgi.py
│   └── wsgi.py
│
└── study/                          # Core application
    ├── models.py                   # All data models (see Data Models section)
    ├── views.py                    # Request handlers — upload, summarize, TTS, translate, quiz, auth
    ├── summarizer.py               # T5 summarization module with fallback logic
    ├── urls.py                     # App-level URL routing
    ├── forms.py                    # Django form definitions
    ├── admin.py                    # Admin site registration
    ├── migrations/                 # Database migration history
    ├── templates/                  # Django HTML templates (9 pages)
    └── static/css/                 # Per-page stylesheets (9 files)
```

### Data Models

| Model | Purpose |
|---|---|
| `CustomUser` | Extended Django user (`AbstractUser`) |
| `Profile` | Per-user study goal and daily available hours |
| `StudyMaterial` | Uploaded PDF metadata, extracted text, summary, processing state |
| `Module` | 10-page content chunk with title, sequence, page range, estimated time |
| `Quiz` | One quiz instance per module |
| `Question` | MCQ or True/False question tied to a quiz |
| `Option` | Answer choice with correctness flag |
| `QuizAttempt` | Score and timestamp for each attempt |
| `UserProgress` | Completed modules, current module, rolling average quiz score |
| `Notification` | Study reminders and milestone alerts |
| `Streak` | Current streak, longest streak, last active date |

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Backend | Django 5.1 | Web framework, ORM, auth |
| Database | SQLite (dev) / PostgreSQL (prod) | Relational data storage |
| ML — Summarization | HuggingFace Transformers, T5-base, PyTorch | Abstractive summary generation |
| ML — NLP | spaCy `en_core_web_sm` | NER-based quiz question generation |
| PDF | PyPDF2 | Text extraction from uploaded PDFs |
| Accessibility | pyttsx3 | Text-to-speech playback |
| Translation | googletrans | Multilingual content translation |
| Frontend | Django Templates, HTML/CSS | Server-rendered UI (9 pages) |

---

## Installation

### Prerequisites

- Python 3.12+
- pip
- (Optional) PostgreSQL for production deployments

### 1. Clone the Repository

```bash
git clone https://github.com/thilak0105/smart-study-helper.git
cd smart-study-helper
```

### 2. Create and Activate a Virtual Environment

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
```

### 3. Install Core Dependencies

```bash
pip install django PyPDF2 pyttsx3 "googletrans==4.0.0rc1" nest_asyncio
```

### 4. Install Optional ML/NLP Dependencies

These are required for T5 summarization and spaCy quiz generation. The app runs without them via fallback logic, but quality degrades.

```bash
pip install transformers torch spacy
python -m spacy download en_core_web_sm
```

### 5. Configure the Database

**SQLite (default — no config needed):**
```bash
python manage.py migrate
```

**PostgreSQL:**
Set the following environment variables, then run migrations:
```bash
export USE_POSTGRES=true
export POSTGRES_DB=study
export POSTGRES_USER=<your_user>
export POSTGRES_PASSWORD=<your_password>
export POSTGRES_HOST=localhost
export POSTGRES_PORT=5432

python manage.py migrate
```

### 6. Run the Development Server

```bash
python manage.py runserver 127.0.0.1:8000
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser.

---

## Usage

1. **Sign up** and log in to your account
2. **Upload a PDF** from your dashboard — the system auto-processes it in the background
3. **Read the summary** as paragraphs or bullet points on the Summary page
4. Press **🔊 Read Aloud** to play the summary via text-to-speech
5. Go to **Notes** for condensed bullet-point takeaways
6. Take the **adaptive quiz** generated from your notes
7. Track your **progress and streaks** on the dashboard

---

## API Reference

All endpoints are relative to the base URL (`http://127.0.0.1:8000` locally).

| Method | Endpoint | Description |
|---|---|---|
| `GET/POST` | `/` | Dashboard / dummy index |
| `GET` | `/home/` | Landing page |
| `GET/POST` | `/login/` | User login |
| `GET/POST` | `/signup/` | User registration |
| `GET` | `/profile/` | User profile and study goal settings |
| `POST` | `/upload_study_material/` | Upload a PDF study material |
| `GET` | `/process_study_material/<id>/` | Trigger summarization pipeline for a material |
| `GET` | `/lessons/<id>/` | Module-by-module lesson view |
| `GET` | `/generate_notes/<id>/` | Generate bullet-point notes from summary |
| `GET` | `/questions_form/` | View AI-generated comprehension questions |
| `POST` | `/submit_answers/` | Submit quiz answers |
| `GET` | `/streaks/` | JSON endpoint — current and longest streak |
| `POST` | `/start_text_to_speech/` | Start TTS playback of current summary |
| `POST` | `/stop_text_to_speech/` | Stop TTS playback |
| `POST` | `/translate_pdf/` | Translate PDF content to target language |

---

## Configuration

Key settings in `studyhelper/settings.py`:

| Setting | Default | Notes |
|---|---|---|
| `DEBUG` | `True` | Set `False` in production |
| `SECRET_KEY` | hardcoded | Move to environment variable before deploying |
| `ALLOWED_HOSTS` | `[]` | Add your domain/IP for production |
| `USE_POSTGRES` | `false` | Set `true` to switch from SQLite to PostgreSQL |
| `MEDIA_ROOT` | `media/` | Uploaded files directory — exclude from version control |

### Production Checklist

- [ ] Move `SECRET_KEY` to an environment variable
- [ ] Set `DEBUG = False`
- [ ] Configure `ALLOWED_HOSTS` with your domain
- [ ] Switch to PostgreSQL via `USE_POSTGRES=true`
- [ ] Set up a production-grade web server (gunicorn + nginx)
- [ ] Add an async task queue (Celery + Redis) for long PDF processing jobs
- [ ] Exclude `media/` and `db.sqlite3` from version control

---

## Troubleshooting

**`ImportError: cannot import name X from transformers`**
Reinstall torch and transformers cleanly. The app will fall back to extractive summarization automatically if Torch is unavailable.

**`spaCy model not found`**
Run `python -m spacy download en_core_web_sm`. The app falls back to heuristic question generation if the model is absent.

**`OperationalError: could not connect to server` (PostgreSQL)**
Either switch back to SQLite (unset `USE_POSTGRES`) or verify your Postgres credentials and ensure the service is running.

**`ModuleNotFoundError: googletrans / pyttsx3 / nest_asyncio`**
```bash
pip install "googletrans==4.0.0rc1" pyttsx3 nest_asyncio
```

---

## Roadmap

- [ ] Async task queue (Celery + Redis) for long PDF processing
- [ ] BERT-based answer evaluation for open-ended quiz responses
- [ ] Pegasus / BART for higher-quality abstractive summarization
- [ ] React frontend with richer interactivity and accessibility controls
- [ ] Spaced repetition system (SRS) for long-term retention
- [ ] Role-based access and instructor analytics dashboard
- [ ] Mobile app with offline PDF support
- [ ] CI pipeline — linting, tests, migration checks
- [ ] REST API schema (OpenAPI / Swagger)

---

## Contributors

| Name | GitHub |
|---|---|
| Thilak L | [@thilak0105](https://github.com/thilak0105) |
| Bharath Kesav R | [@bk1210](https://github.com/bk1210) |
| Subramanian G | [@Demoncyborg07](https://github.com/Demoncyborg07) |
| Raghul A R | [@a-steel-heart](https://github.com/a-steel-heart) |

---

<div align="center">

*Built to make learning accessible for everyone.*

</div>
