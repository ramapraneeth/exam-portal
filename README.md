# Mock Test Portal (AP EAPCET-style)

A Django-based online mock test / exam platform with a candidate login screen
and exam interface styled after AP EAMCET/EAPCET style mock test sites:
candidate photo, system name, countdown timer, subject tabs, and a
color-coded question palette (Answered / Not Answered / Not Visited /
Marked for Review / Answered & Marked for Review).

## Features

- Candidate login (ID + password)
- Tests can be **single-subject** or **multi-subject** (e.g. Maths only, or
  Maths + Physics + Chemistry combined) — configured per test, no code changes needed
- Exam interface: countdown timer with auto-submit, subject tabs, question
  palette with live status colors, Save & Next / Clear Response / Mark for
  Review & Next / Submit
- Auto-grading with configurable marks and negative marking per question
- Section-wise results breakdown
- Full Django admin panel to create subjects, tests, sections, questions,
  and choices — no custom admin UI needed to get started
- SQLite by default (zero setup) — swap to PostgreSQL for production in one settings change

## 1. Requirements

- Python 3.10+
- pip

## 2. Setup (run these on your own machine — this environment has no internet access to install packages)

```bash
# from inside the exam_portal/ folder
python -m venv venv
source venv/bin/activate        # on Windows: venv\Scripts\activate

pip install -r requirements.txt

python manage.py migrate
python manage.py createsuperuser   # create your admin login

# Optional but recommended: load demo data (1 candidate + 2 sample tests)
python manage.py seed_demo_data

python manage.py runserver
```

Visit:
- **Candidate login:** http://127.0.0.1:8000/
- **Admin panel:** http://127.0.0.1:8000/admin/

If you ran `seed_demo_data`, log in as a candidate with:
- **Candidate ID:** `11111`
- **Password:** `pass123`

## 3. Adding your own tests and questions

Everything is manageable from the Django admin (`/admin/`):

1. **Subjects** → add e.g. Mathematics, Physics, Chemistry, English, Reasoning, etc.
2. **Tests** → create a test (title, duration, instructions). On the same page,
   add one or more **Test Sections** inline — one section per subject.
   - 1 section = single-subject test
   - 3 sections = multi-subject test (tabs shown in exam UI)
3. **Questions** → go to the Questions section, pick the relevant Test Section,
   write the question text, and add 4 choices inline, marking the correct one.
   Set `marks` and `negative_marks` per question if you want negative marking.
4. **Candidates** → create a normal Django **User** (`username` = candidate's
   login ID), then create a matching **Candidate Profile** with the same
   `candidate_id` and an optional photo upload.

No coding needed for day-to-day content management — that's the point of
using Django's built-in admin.

## 4. How the candidate-facing flow works

1. Candidate logs in with their ID and password (`login.html`)
2. Sees a list of available tests (`test_list.html`)
3. Reads instructions, clicks "I am ready to begin" → starts the attempt
4. Takes the test (`take_test.html` + `exam.js`):
   - Countdown timer runs client-side and auto-submits when it hits zero
   - Clicking a subject tab loads that subject's first question
   - Answers are saved via AJAX as the candidate navigates (no full page reloads)
   - The right-hand palette grid updates colors live
5. Submits (or is auto-submitted) → redirected to the results page showing
   score and a section-wise breakdown

## 5. Customizing the look

All styling lives in `static/css/style.css`, organized into clearly labeled
sections (Login Page, Dashboard, Instructions, Exam Page, Result Page).
Colors are simple hex values near the top of each block if you want to match
a different color scheme exactly.

The JS logic lives in `static/js/exam.js` — it's plain vanilla JS (no
framework), so it's easy to read and modify.

## 6. Important before going live (production checklist)

This project ships with development-friendly defaults. Before deploying for
real candidates, you should:

- [ ] Set `DEBUG = False` in `exam_portal/settings.py`
- [ ] Replace `SECRET_KEY` with a new, secret value (don't reuse the placeholder)
- [ ] Set `ALLOWED_HOSTS` to your actual domain(s), not `['*']`
- [ ] Switch the database from SQLite to PostgreSQL for concurrent candidates:
      update the `DATABASES` setting in `settings.py` and add `psycopg2-binary`
      to `requirements.txt`
- [ ] Set up proper static file serving (e.g. WhiteNoise, or your host's
      static file handling) since Django doesn't serve static files
      efficiently in production by itself
- [ ] Consider moving the countdown timer's authoritative time check fully
      server-side (the current `time_left_seconds` on `TestAttempt` is already
      server-calculated and re-validated on submit, but review this against
      your integrity requirements if this is a high-stakes exam)
- [ ] Add HTTPS (most hosts like Railway/Render handle this automatically)

## 7. Deployment

This is a standard Django project, so it deploys the same way to:
- **Railway** or **Render** (easiest — both auto-detect Django, just add a
  `Procfile` with `web: gunicorn exam_portal.wsgi` and add `gunicorn` to
  requirements.txt)
- **PythonAnywhere** (good free tier for small-scale use)
- Any VPS (DigitalOcean, AWS EC2, etc.) with gunicorn + nginx

If you'd like, I can prepare deployment-specific files (Procfile, production
settings split, etc.) for whichever platform you choose.

## Project structure

```
exam_portal/
├── manage.py
├── requirements.txt
├── exam_portal/          # project settings, root urls
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── exams/                 # the main app
│   ├── models.py          # Subject, Test, TestSection, Question, Choice,
│   │                       # CandidateProfile, TestAttempt, Answer
│   ├── admin.py            # admin panel configuration
│   ├── views.py             # login, test list, exam logic, results
│   ├── forms.py
│   ├── urls.py
│   ├── templates/exams/    # login.html, test_list.html, instructions.html,
│   │                        # take_test.html, result.html
│   └── management/commands/seed_demo_data.py
├── static/
│   ├── css/style.css
│   └── js/exam.js
└── media/candidate_photos/   # uploaded candidate photos
```
