# JobTrack AI — AI-Powered Job Application Tracker

A Django app for tracking job applications - add jobs, move them through a
status pipeline (Wishlist → Applied → Screening → Interview → Selected /
Rejected), schedule interviews, search/filter your list, and get an AI
breakdown of a job description (summary, required skills, required
experience, important technologies, interview prep). Module 20 assignment.

## Features

- User registration, login, logout (Django's built-in auth). Every user only
  ever sees and manages their own applications - everything is filtered by
  `request.user`.
- Full CRUD on job applications: create, list, detail view, edit, delete.
- Search by job title/company, filter by status, location, and category.
- Categories (one per job) and Tags (many per job) - basic FK / M2M models.
- Interviews tied to a specific application (date/time, type, meeting link,
  notes). One application can have several interviews.
- AI Job Description Analyzer - paste a job description, send it to an AI
  model (via OpenRouter), and get back a Job Summary, Required Skills,
  Required Experience, Important Technologies, and Interview Preparation
  Suggestions.
- Dashboard - total applications, applications by status, recent
  applications, upcoming interviews. All calculated with Django ORM
  aggregates (`Count`), not manual counting in Python.
- Bootstrap 5 (CDN, no build step) for styling.

## Tech Stack

- Python 3.11 / Django 4.2
- SQLite (Django's default database)
- Bootstrap 5 (CDN)
- [OpenRouter](https://openrouter.ai/) for the AI API call (OpenAI-compatible
  API, has free models so this doesn't cost anything to run/test)
- `python-decouple` for reading settings from `.env`

## Project Structure

```
ai-job-application-tracker/
├── jobtracker/          # Django project settings/urls
├── tracker/             # the actual app
│   ├── models.py        # Category, Tag, JobApplication, Interview, JobAnalysis
│   ├── views.py         # all the views (function-based)
│   ├── forms.py         # register form, application form, interview form
│   ├── ai_analyzer.py   # the OpenRouter API call + response parsing
│   ├── urls.py
│   ├── admin.py
│   ├── management/commands/seed_data.py   # seeds demo data
│   └── templates/tracker/
│       ├── dashboard.html            # required page: Dashboard
│       ├── application_list.html     # required page: Application List
│       ├── application_detail.html   # required page: Application Details
│       ├── application_form.html     # required page: Create/Edit Application
│       ├── analysis_detail.html      # required page: AI Analysis
│       ├── login.html / register.html
│       ├── interview_form.html, analyze_confirm.html, application_confirm_delete.html
├── manage.py
├── requirements.txt
├── .env.example         # copy to .env and fill in your own key
└── db.sqlite3           # created after running migrate
```

## Models

- **Category** - a broad bucket for a job (e.g. "Backend", "Data"). One
  category per application (`ForeignKey`).
- **Tag** - freeform labels (e.g. "Remote", "Django"). Many per application
  (`ManyToManyField`).
- **JobApplication** - the main model. `ForeignKey` to Django's built-in
  `User` (`related_name='applications'`, so `user.applications.all()` works).
  Holds title, company, description, location, salary, URL, application
  date, status, notes.
- **Interview** - `ForeignKey` to `JobApplication`
  (`related_name='interviews'`). One application can have several
  interviews.
- **JobAnalysis** - `OneToOneField` to `JobApplication`. Stores the AI
  analyzer's result so it doesn't have to call the API again every page
  load; re-running the analysis overwrites it.

## Setup Instructions

1. Clone the repo and go into it:
   ```
   git clone https://github.com/<your-username>/ai-job-application-tracker.git
   cd ai-job-application-tracker
   ```

2. Create a virtual environment and install dependencies:
   ```
   python -m venv venv
   source venv/bin/activate      # on Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Set up your `.env` file (see the section below - **do this before running
   anything**, the AI feature won't work without it, but everything else
   will run fine even with a blank key):
   ```
   cp .env.example .env          # Mac/Linux/Git Bash/PowerShell
   copy .env.example .env        # Windows Command Prompt (cmd.exe) - cp doesn't exist there
   ```
   Then open `.env` and fill in your own `OPENROUTER_API_KEY`.

4. Run migrations:
   ```
   python manage.py migrate
   ```

5. (Optional) Seed sample data so you have something to look at / screenshot:
   ```
   python manage.py seed_data
   ```
   This creates a `demo_user` account (password: `demo12345`) with 4 sample
   applications and 1 upcoming interview.

6. Run the server:
   ```
   python manage.py runserver
   ```

7. Open `http://127.0.0.1:8000/` in your browser. You'll be redirected to
   login. Use the seeded `demo_user` account, or click "Register here" to
   make your own.

### Admin login (optional, for checking the database directly)

```
python manage.py createsuperuser
```
then visit `http://127.0.0.1:8000/admin/`.

## Pages

| Page | URL |
|---|---|
| Registration | `/register/` |
| Login | `/login/` |
| Dashboard | `/` |
| Application List (search/filter) | `/applications/` |
| Application Details | `/applications/<id>/` |
| Create Application | `/applications/new/` |
| Edit Application | `/applications/<id>/edit/` |
| **AI Analysis** | `/applications/<id>/analysis/` |

The "Analyze with AI" button on an application's detail page takes you to
`/applications/<id>/analyze/` (a confirm step showing the job description
about to be sent), which then redirects to the dedicated AI Analysis page
above once the result is ready.

## Setting up the API key (.env)

This app calls [OpenRouter](https://openrouter.ai/) for the AI Job
Description Analyzer feature. **Never commit a real API key to GitHub** -
that's an explicit requirement of this assignment, and it's just good
practice generally.

1. Sign up at [openrouter.ai](https://openrouter.ai) and generate a key at
   [openrouter.ai/keys](https://openrouter.ai/keys).
2. Copy `.env.example` to `.env` (this file is in `.gitignore`, so it never
   gets pushed). The command differs by terminal:
   ```
   cp .env.example .env          # Mac/Linux/Git Bash/PowerShell
   copy .env.example .env        # Windows Command Prompt (cmd.exe)
   ```
   `cp` is a Mac/Linux command and doesn't exist in plain Windows Command
   Prompt - that's the most common reason this step "doesn't work." If
   neither command runs, just duplicate `.env.example` in your file manager
   and rename the copy to `.env`.
3. Open `.env` and paste your key in:
   ```
   OPENROUTER_API_KEY=sk-or-v1-your-actual-key-here
   ```
4. `OPENROUTER_MODEL` is also set in `.env`, defaulting to
   `openai/gpt-oss-20b:free` - a free model, so testing this feature costs
   nothing. OpenRouter's free model list changes over time (models get
   added/deprecated every few weeks), so check
   [openrouter.ai/models](https://openrouter.ai/models) and filter for
   `:free` if this one stops working, then update the line in `.env`. No
   code changes needed to switch models.
5. **Built-in fallback:** free models on OpenRouter share a request pool
   with everyone else using them, so they can occasionally return a
   temporary rate-limit error that has nothing to do with your own usage.
   If the model set in `.env` fails, `tracker/ai_analyzer.py` automatically
   retries once with a second free model
   (`nvidia/nemotron-3-nano-30b-a3b:free`) before giving up - so a single
   rate-limited request usually doesn't stop the feature from working.

**How the app reads it:** `jobtracker/settings.py` uses `python-decouple`'s
`config()` function to read `OPENROUTER_API_KEY` and `OPENROUTER_MODEL` out
of `.env` at startup. `tracker/ai_analyzer.py` then reads them off
`django.conf.settings` when it makes the API call. If `.env` is missing the
key, the analyzer view shows an error message instead of crashing.

### What's actually happening in the AI API call

(This is `tracker/ai_analyzer.py`, function `analyze_job_description`.)

1. **Build the request body.** OpenRouter (and OpenAI) expect a JSON object
   with a `model` name and a `messages` list. Each message has a `role`
   (`system` or `user`) and `content`. The `system` message is instructions
   for how the model should behave; the `user` message is the actual input -
   in this case, the pasted job description.
2. **Ask for a specific output shape.** The system prompt tells the model to
   reply with *only* a JSON object containing exactly 5 keys (`summary`,
   `required_skills`, etc.), no extra text. This is the whole trick to
   getting structured data back from a model that normally just writes
   paragraphs - you describe the shape you want and hope it follows
   instructions (it usually does, but not always perfectly, which is why the
   next steps exist).
3. **Send the HTTP request.** `requests.post()` sends the JSON body to
   OpenRouter's `/chat/completions` endpoint with the API key in the
   `Authorization` header. This is a normal HTTP POST - no special SDK
   magic, just JSON over HTTPS.
4. **Handle network failures.** `response.raise_for_status()` raises an
   exception if OpenRouter returns an error status (rate limit, bad key,
   etc.), caught so the view can show a friendly message instead of a raw
   Python traceback.
5. **Pull the text out of the response.** The response JSON has a
   `choices` list (usually just one choice) with a nested
   `message.content` string - that's where the model's actual reply text
   lives.
6. **Clean and parse the reply.** Sometimes a model wraps its JSON in
   ` ```json ... ``` ` code fences even when told not to - stripped off
   before parsing. Then `json.loads()` turns the text into a Python dict.
7. **Fill in fallback values.** If the model somehow left out a key,
   `.get(key, 'Not provided.')` fills in a placeholder instead of crashing
   with a `KeyError`.
8. **Save the result.** Back in `views.py`, the parsed dict is saved via
   `JobAnalysis.objects.update_or_create(...)` - so re-running the analysis
   updates the existing row instead of creating duplicates.
9. **Fallback on failure.** `analyze_job_description()` wraps the actual API
   call (now a separate helper, `_call_openrouter()`) so it can call it
   twice: once with whatever model is set in `.env`, and if that fails for
   any reason (rate limit, bad response, timeout), once more with a fixed
   backup free model. Only if both attempts fail does the view show an
   error message.

## Notes / Assumptions

- Status is a plain `CharField` with `choices` - the assignment's sequence
  (Wishlist → Applied → Screening → Interview → Selected/Rejected) is the
  order shown in the dropdown, but the database doesn't *enforce* moving
  through it in order. A user can jump straight from Wishlist to Rejected -
  that's realistic (a job can be rejected at any stage) and keeping the
  field a plain choice list is simpler than building a state machine for a
  student assignment.
- `application_date` is optional (`null=True, blank=True`) since it makes
  sense to add a job to "Wishlist" before actually applying.
- Search matches job title OR company name (`icontains`, case-insensitive)
  using Django's `Q` objects to OR the two conditions in a single query.
- The AI analysis result is cached in the `JobAnalysis` model instead of
  being re-fetched from the API on every page load - saves API calls and
  means the page still shows the last analysis even if the API is down.
- Dates in the seed data are relative to "today" (`timezone.now()`), so the
  upcoming interview always shows up as upcoming no matter when you run the
  seed command.

## Django & AI Integration Concepts Used

- **Models & Django ORM** - `JobApplication`, `Category`, `Tag`,
  `Interview`, `JobAnalysis`, all defined as `models.Model` subclasses.
- **Model relationships** - `ForeignKey` (`JobApplication.user`,
  `Interview.application`, `JobApplication.category`), `ManyToManyField`
  (`JobApplication.tags`), `OneToOneField` (`JobAnalysis.application`), and
  `related_name` used throughout (`applications`, `interviews`) so reverse
  lookups read naturally (`user.applications.all()`).
- **Migrations** - `makemigrations` / `migrate`.
- **CRUD operations** - full create/list/detail/edit/delete flow for
  `JobApplication`.
- **Forms / ModelForms** - `JobApplicationForm`, `InterviewForm`,
  `RegisterForm` (subclasses Django's built-in `UserCreationForm`).
- **Authentication & Authorization** - `login()`, `logout()`,
  `authenticate()`, `@login_required`, and every query scoped to
  `request.user` so users can only ever see/edit their own data (verified:
  visiting another user's application by ID returns a 404, not just a
  blocked page).
- **Search & filtering with the ORM** - `Q` objects for OR'd search,
  `.filter()` chains for status/location/category, `icontains` for
  case-insensitive partial matches. No manual Python loops over querysets.
- **ORM aggregates** - `.values('status').annotate(total=Count('id'))` for
  the dashboard's "applications by status" breakdown, instead of counting
  in a Python loop.
- **Templates** - Django template language, template inheritance
  (`{% extends %}` / `{% block %}`), `{% for %}` / `{% if %}`, template
  filters (`|date`, `|default`).
- **Django admin** - all 5 models registered for direct inspection.
- **External AI API integration** - `requests` library, structured prompts,
  JSON parsing, error handling for a third-party HTTP API (OpenRouter).
- **Environment variables / secrets management** - `python-decouple`
  reading `SECRET_KEY` and `OPENROUTER_API_KEY` from a gitignored `.env`
  file, never hardcoded.
