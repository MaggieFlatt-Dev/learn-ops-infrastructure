# learn-ops-api: AI-Assisted Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
| `LearningAPI` | The core Django app containing the project's domain logic — models, views, serializers, migrations, tests, and fixtures for cohorts, students, courses, assessments, skills, etc. |
| `LearningPlatform` | The Django *project* package — holds global `settings.py`, top-level `urls.py`, and `wsgi.py`, which wire all the apps together into one runnable site. |
| `LogViewer` | A small secondary Django app that provides a web UI (with its own templates, urls, and views) for browsing the application's log files. |
| `config` | Deployment/server configuration that lives outside Django itself: nginx configs for the API and client, and a YAML config for the platform. |
| `static` | Source static assets (CSS/JS/images) referenced by templates before they're collected for production. |
| `staticfiles` | The output of Django's `collectstatic` command — combined static files (admin, DRF browsable API styling) ready to be served by nginx/whitenoise. |
| `templates` | Project-level HTML templates, e.g. overrides for the Django admin site. |
| `logs` | Log files written by the running application (e.g. `learning_platform.json`). |
| `.github` | CI/CD workflow definitions (GitHub Actions) — e.g. `main.yml`, `collectstatic.yml`, `seed.yml` — for running tests, collecting static files, and seeding data. |
| `.vscode` | Editor-specific settings and launch configs for VS Code. |
| `.git` | Git repository metadata (version history, branches, etc.) for this project. |

## 2. Folders inside `LearningAPI`

| Folder | What responsibility does it own and why? |
|--------|------------------------------------------|
| `migrations` | Django-generated database migration files that record every schema change over time, so the DB schema can be created/updated reproducibly. |
| `tests` | The automated test suite (e.g. `test_cohort.py`, `test_course.py`) that verifies API endpoints and business logic behave correctly. |
| `models` | The app's data models, organized into subpackages by domain: `coursework` (courses, projects, capstones, learning objectives), `people` (users, cohorts, assessments, mentors, teams), and `skill` (skill records, learning records, weights), plus a shared `tag.py`. |
| `serializers` | DRF serializer classes that convert model instances to/from JSON for API requests and responses (e.g. `cohort_serializer.py`, `user_serializer.py`). |
| `fixtures` | JSON fixture files used to pre-populate the database with sample/seed data (users, cohorts, courses, assessments, etc.) for development and testing. |
| `views` | DRF view/viewset classes that implement the API endpoints — mostly one file per resource (e.g. `course_view.py`, `cohort_view.py`), plus a `github` subpackage and auth-related views (`auth.py`, `github_login.py`). |

## 3. What is the Pipfile?

The `Pipfile` is the dependency manifest for `pipenv`, Python's equivalent of `package.json` for Node. It exists so the project's Python dependencies — and the exact Python version it needs — are declared in one place and can be reproduced consistently across machines (a teammate's laptop, CI, or a Docker build).

It's organized into a few sections:

- **`[[source]]`** — where packages are downloaded from (PyPI).
- **`[packages]`** — production dependencies needed to run the app, e.g. `django`, `djangorestframework` (the API framework), `gunicorn` (production server), `psycopg2-binary`/`dj-database-url` (Postgres support), `django-allauth`/`dj-rest-auth` (authentication), `valkey` (Redis-compatible client), and several logging/observability libs (`structlog`, `django-structlog`, `python-json-logger`, `opensearch-py`, `django-prometheus`).
- **`[dev-packages]`** — tools only needed during development/testing, e.g. `pylint` (linting), `pytest`/`pytest-django`/`pytest-cov` (testing), `debugpy` (debugger). These aren't installed in production.
- **`[requires]`** — pins the required Python version (`3.11.11`), so everyone uses the same interpreter version.
- **`[scripts]`** — shortcut commands (here, `migrate` runs `python3 manage.py migrate`) that can be run via `pipenv run migrate`.

Alongside it, `Pipfile.lock` records the exact resolved versions (and hashes) of every dependency — including transitive ones — so installs are fully reproducible, not just "whatever satisfies `*` today".


## 4. Key packages

| Package | What functionality does it provide and why? |
|---------|---------------------------------------------|
| `django` | The core web framework. It provides the ORM (the `models` in `LearningAPI`), the admin site (`django.contrib.admin`), URL routing, the settings system, and the request/response handling that everything else in the project plugs into. This project depends on it because it's the foundation the whole API is built on top of — without it there's no app, no models, no migrations. |
| `djangorestframework` (`rest_framework`) | Adds the tools to build a REST API on top of Django: serializers (turning model instances into JSON and back), generic views/viewsets, routers, and token authentication (`rest_framework.authtoken`). The project depends on it because `LearningAPI` is fundamentally an API — its `views` and `serializers` directories are built directly on DRF's classes, and `DEFAULT_AUTHENTICATION_CLASSES` in settings uses DRF's `TokenAuthentication`. |
| `django-allauth` (`allauth`) | Handles authentication and account management beyond Django's built-in basics — user registration, email verification, and social/OAuth login. In this project it's configured with `allauth.socialaccount.providers.github`, which is what powers "Login with GitHub" (used together with `dj_rest_auth` to expose those flows as API endpoints, matching `views/github_login.py` and `views/github/`). The project depends on it because students/staff log in via their GitHub accounts rather than a custom username/password system. |

## 5. What does `decorators.py` do?

A **decorator** is a function that wraps another function to add behavior to it without changing the wrapped function's own code. In Python, `@some_decorator` placed above a function definition is shorthand for `some_function = some_decorator(some_function)` — the decorator returns a new function that typically does something extra (checks, logging, etc.) and then calls the original.

`LearningAPI/decorators.py` defines two permission-checking decorators, `is_instructor()` and `is_staff()`. Each one:

1. Is a "decorator factory" — calling `is_instructor()` returns the actual `decorator` function.
2. That `decorator` wraps a view method in `__wrapper`, which checks `request.user.groups.filter(name='Instructors').exists()` (or `'Staff'` for `is_staff()`).
3. If the user belongs to that group, it calls the original view function as normal. If not, it short-circuits and returns a `401 Unauthorized` response with a message like `"You must be an instructor"`.

In practice they're applied to view methods with Django's `method_decorator`, e.g. in `views/student_assessment.py`:

```python
@method_decorator(is_instructor())
def create(self, request):
    ...
```

This means the `create` action on `StudentAssessmentView` only runs its real logic if the requesting user is in the "Instructors" group — otherwise the request is rejected before any business logic executes. This keeps authorization checks reusable and out of the main view logic, instead of repeating the same group-check `if` statement in every view that needs it.

## 6. What is a serializer, and how does it fit the request/response cycle?

A **serializer** (from Django REST Framework) is a class that converts between complex Python objects — like Django model instances — and simple data formats such as JSON, and vice versa. It's the translation layer between the database/ORM world and the "wire format" that an API client (the React frontend, mobile app, etc.) sends and receives.

For example, `serializers/cohort_serializer.py`:

```python
from rest_framework import serializers
from LearningAPI.models import Cohort


class CohortSerializer(serializers.ModelSerializer):
    class Meta:
        model = Cohort
        fields = '__all__'
```

`CohortSerializer` is a `ModelSerializer` tied to the `Cohort` model. Because it lists `fields = '__all__'`, it automatically generates a field for every column on the `Cohort` model (name, dates, active flag, etc.), and knows how to validate and save data back to that model.

How it fits into the request/response cycle:

- **Incoming request (deserialization):** A view receives a request — say a POST with JSON data for a new cohort. It passes `request.data` into `CohortSerializer(data=request.data)`. The serializer validates the JSON (correct types, required fields) and, if valid, can create or update a `Cohort` model instance via `serializer.save()`.
- **Outgoing response (serialization):** When a view fetches one or more `Cohort` objects from the database, it passes them to `CohortSerializer(cohort)` or `CohortSerializer(cohorts, many=True)`. Calling `.data` on the serializer turns those model instances into plain Python dicts/lists, which DRF's `Response` then renders as JSON for the client.

So the flow is roughly: **HTTP request → view → serializer (validate/deserialize) → model/DB → serializer (serialize) → view → HTTP response (JSON)**.

## 7. One model and what it represents

A **Django model** is a Python class that defines the structure of a database table — each class attribute (e.g. `models.CharField`, `models.DateField`, `models.BooleanField`) becomes a column, and each instance of the class represents one row. Django's ORM uses these classes to generate SQL automatically (via migrations) and lets you query/create/update rows using Python instead of raw SQL.

The `models` folder is split into subpackages by domain — `coursework`, `people`, and `skill` — each containing one model per file.

**Example: `Cohort`** (`models/people/cohort.py`)

```python
class Cohort(models.Model):
    """Model for student cohorts"""
    name = models.CharField(max_length=55, unique=True)
    slack_channel = models.CharField(max_length=55, unique=False)
    start_date = models.DateField(auto_now=False, auto_now_add=False)
    end_date = models.DateField(auto_now=False, auto_now_add=False)
    break_start_date = models.DateField(auto_now=False, auto_now_add=False)
    break_end_date = models.DateField(auto_now=False, auto_now_add=False)
    active = models.BooleanField(default=False)
```

**Real-world thing it represents:** A `Cohort` is a group of students going through the program together — e.g. "Day Cohort 65" — with a name, an associated Slack channel, a start/end date, a break period, and whether it's currently active.

**Why the API needs to track this data:** The whole platform is organized around cohorts — students belong to a cohort (`NssUserCohort`), instructors/coaches are assigned to cohorts, course schedules and events (`CohortEvent`, `CohortCourse`) are tied to a cohort's timeline, and the `is_active_on_date()` method lets the system figure out, for any given date, whether a cohort is currently in session (vs. on break or finished). Without a `Cohort` model, there'd be no way to group students, schedule curriculum, or scope dashboards/reports to "everyone currently in this class."


## 8. Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View (function-based, `@api_view`) | `notify()` in `views/notify.py` | For a one-off action that doesn't map to CRUD on a single model — here, sending a Slack notification. It's wired to a single URL (`path('notify', views.notify, ...)`) and only handles `POST`. |
| ViewSet | `CohortViewSet` in `views/cohort_view.py` | For a resource that needs the full set of CRUD operations (list, retrieve, create, update, destroy) plus extra custom actions, all grouped under one URL prefix. It's registered with the router (`router.register(r'cohorts', views.CohortViewSet, 'cohort')`), which automatically generates `/cohorts`, `/cohorts/<pk>`, etc. |

**The difference:**

- A **plain view** (a function decorated with `@api_view`, or a class based on `APIView`) handles one specific endpoint and you write the logic for whatever HTTP methods it needs. `notify()` is a single function that only responds to `POST /notify` — there's no "list notifications" or "delete a notification" because that concept doesn't make sense here. It's mapped to its URL explicitly with `path(...)`.
- A **ViewSet** (e.g. `ViewSet` or `ModelViewSet`) groups together the standard operations for a *resource* — `create`, `list`, `retrieve`, `update`, `destroy` — as methods on one class, plus any custom `@action` methods. Instead of writing a `path()` for each operation, you `router.register()` the viewset once and DRF's router generates all the conventional URLs (`GET /cohorts`, `POST /cohorts`, `GET /cohorts/<id>`, `PUT /cohorts/<id>`, `DELETE /cohorts/<id>`) and maps them to the matching methods automatically.

**When to choose one over the other:**

- Choose a **plain view/function** when the endpoint represents an *action* rather than a *resource* — something that doesn't fit neatly into "list/create/retrieve/update/delete" (e.g. sending a notification, returning a computed report like `popular_queries`, handling a login callback).
- Choose a **ViewSet** when you're exposing a database model (or model-like resource) through the API and want the standard CRUD operations — and possibly a few extra related actions (e.g. `CohortViewSet` likely has custom actions like `assign` or `migrate` alongside `create`/`list`/etc., as hinted by its permission class). This avoids repetitive URL wiring and keeps all operations on that resource in one place.

## 9. What replaces templates and why?

In Django's Model-Template-View pattern, the **template** is responsible for taking data and rendering it into a response — normally an HTML page that gets sent to a browser.

In this project, **serializers take over that "render the response" role**, but instead of producing HTML, they produce JSON. A serializer like `CohortSerializer` (see #6) defines exactly which fields of a `Cohort` model get turned into a response, the same way an HTML template would define which fields get dropped into `<div>`s and `<span>`s. The view still does the same job as in MTV — fetch/update data via the model — but the final "shape the output" step is handed to a serializer instead of a `.html` template file.

The few `templates/` directories that *do* exist in this project (`learn-ops-api/templates`, `LogViewer/templates`) aren't part of the API itself — they're for the Django **admin site** and the internal **log viewer** tool, which are small server-rendered web pages for staff, separate from the public REST API.

**Why this makes sense for a REST API:**

- A REST API's clients are other programs (the React frontend in `learn-ops-client`, scripts, mobile apps), not browsers expecting a styled page. Those clients want structured data they can parse and render themselves, not pre-built HTML.
- Separating "data" (JSON from the API) from "presentation" (HTML/CSS in the frontend) lets the frontend and backend be developed, deployed, and scaled independently — the API doesn't need to know or care how the data is displayed.
- JSON is a much more general, lightweight format than HTML — easy for any client (web, mobile, another service) to consume, whereas HTML templates are tightly coupled to "render this in a browser."
- Serializers also handle the *reverse* direction (turning incoming JSON back into model data), which a template can't do — templates are output-only. So serializers fill the role of "shape the data for the client" in both directions, which is exactly what a REST API needs.