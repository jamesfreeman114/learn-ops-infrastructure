# learn-ops-api: AI-Assisted Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
| `LearningAPI` | The main Django app — holds the models, views, serializers, and migrations that implement the actual learning-platform business logic and API endpoints. |
| `LearningPlatform` | The Django *project* package (created by `django-admin startproject`). It holds `settings.py`, root `urls.py`, and `wsgi.py` — the glue that configures and wires all the installed apps together. |
| `LogViewer` | A second, smaller Django app dedicated to a single concern: letting users view the application's log files through the web UI. Kept separate from `LearningAPI` because it's an unrelated responsibility. |
| `config` | Deployment/infrastructure configuration that isn't Python code — nginx config files and a YAML file used when running or deploying the API server. |
| `static` | Source static assets (CSS/JS/images) that Django apps ship with — mostly the Django admin's own static files here. |
| `staticfiles` | The output folder Django's `collectstatic` command writes to — it gathers static files from every app into one place so a production web server can serve them directly. |
| `templates` | Server-rendered HTML templates — here mainly overrides/assets for the Django admin site. |
| `logs` | Where the running server actually writes its rotating log files (e.g. `learning_platform.json`, `.1`, `.2`). |
| `.github` | GitHub-specific configuration — contains `workflows/` for GitHub Actions (CI/CD automation). |
| `.vscode` | Editor configuration for VS Code (launch configs, recommended extensions) — improves the dev experience but isn't part of the running app. |
| `.git` | Git's internal repository data — tracks version history; every git repo needs this. |

## 2. Folders inside `LearningAPI`

| Folder | What responsibility does it own and why? |
|--------|------------------------------------------|
| `views` | The "controller" layer — one file per resource (courses, cohorts, capstones, etc.) containing the functions/classes that receive HTTP requests and return responses. Split into files instead of one giant `views.py` because there are many resources. |
| `serializers` | The translation layer between Django model instances and JSON. Serializers convert models to JSON for responses and validate/parse incoming JSON into Python data for writes — this is what lets DRF views talk to models without hand-rolling JSON parsing. |
| `models` | The database schema, defined as Django ORM classes. Split into subpackages (`coursework`, `people`, `skill`) rather than one `models.py` because the app's data model is large enough that grouping by domain area keeps it navigable. |
| `migrations` | Auto-generated, ordered files describing incremental changes to the database schema over time. Django uses these to create/update the actual database tables to match the models. |
| `tests` | Automated test cases that verify the app's behavior (e.g. `test_cohort.py`, `test_course.py`) so changes can be checked for regressions. |
| `fixtures` | JSON files with sample/seed data (e.g. `LearningAPI_course.json`) that can be loaded into the database with `loaddata` — used for local development, demos, and test setup. |

## 3. What is the Pipfile?

The `Pipfile` is this project's dependency manifest for `pipenv` (a Python packaging tool that combines `pip` and virtual environments). It lists the Python packages the project needs to run, split into `[packages]` (production/runtime dependencies) and `[dev-packages]` (tools only needed while developing, like `pytest` and `pylint`), pins the Python version to use (`3.11.11`), and can define convenience `[scripts]` (here, a `migrate` shortcut for `python3 manage.py migrate`). It exists so that anyone setting up the project — or a deployment pipeline — installs the exact same set of packages, instead of relying on whatever happens to already be on their machine. It's the Python-world equivalent of a `package.json` in a Node/JS project, if that comparison helps.

## 4. Key packages

| Package | What functionality does it provide and why? |
|---------|---------------------------------------------|
| django | The core web framework itself — the ORM (turns Python model classes into database tables/queries), the URL router, the admin site, and the request/response machinery everything else in this project is built on top of. Without it, none of the app's Python code has anything to run inside. |
| djangorestframework | Sits on top of Django to make building a JSON API straightforward: serializers (converting model instances to/from JSON), generic/class-based API views, authentication/permission classes, pagination, etc. `settings.py` configures its `TokenAuthentication`, `IsAuthenticated` permission, and pagination defaults (lines ~145-153), and it's what `LearningAPI/serializers` and `LearningAPI/views` are built with. This project needs it because `LearningAPI` is an API backend (for a separate frontend client), not a server-rendered site — plain Django alone doesn't give you JSON in/out for free. |
| django-allauth | Provides authentication, including third-party/social login. In `settings.py`, `allauth.socialaccount.providers.github` is installed and `SITE_ID`/`SOCIALACCOUNT_LOGIN_ON_GET` are configured, which means this project uses allauth specifically to let users log in via their GitHub account rather than (or in addition to) a plain username/password. It's paired with `dj-rest-auth`, which adapts allauth's login flows into REST API endpoints that a frontend can call. |

## 5. What does `decorators.py` do?

A decorator is a function that wraps another function to add behavior to it, without changing the wrapped function's own code. In Python, `@some_decorator` above a function definition is shorthand for `func = some_decorator(func)` — it swaps the function out for a new one that does something extra and then (usually) calls the original.

`LearningAPI/decorators.py` defines two authorization decorators, `is_instructor()` and `is_staff()`. Each one returns a `decorator` function, which itself returns a `__wrapper` function — this "function that returns a function that returns a function" pattern is what lets the decorator be called with `()`, e.g. `@is_instructor()` instead of `@is_instructor`. When a decorated view method runs, `__wrapper` checks whether `request.user` belongs to the required group (`'Instructors'` or `'Staff'`); if so, it calls the real view method as normal, and if not, it short-circuits and returns a `401 Unauthorized` response instead. This project uses them (via `django.utils.decorators.method_decorator`, e.g. in `views/course_view.py`'s `create`/`update`/`destroy` methods) to enforce "only instructors can do this" rules in one reusable place, instead of repeating the same `if` check inside every view method that needs it.

## 6. What is a serializer, and how does it fit the request/response cycle?

A serializer is a class that converts between two shapes of data: Django model instances/querysets (Python objects backed by the database) and JSON (plain text that can travel over HTTP). It works in both directions — "serializing" turns a model instance into JSON for a response, and "deserializing"/validating turns incoming JSON from a request body into clean, validated Python data that can be saved to a model.

For example, `NssUserSerializer` (in `LearningAPI/serializers/nssuser_serializer.py`) is a `ModelSerializer` tied to the `NssUser` model, exposing only the fields listed in `Meta.fields` (`url`, `slack_handle`, `github_handle`, `mentor`, `user`) — it decides what the outside world is allowed to see of that model, and in what shape.

In the request/response cycle, the serializer sits between the view and the model:
1. A request comes in (e.g. `POST /api/nssusers/`) and DRF routes it to a view.
2. **Incoming data:** the view hands the request body to a serializer, which validates it (e.g. rejects a malformed value) and turns it into data the view can save via the model.
3. The view saves that through the model, which persists it to the database.
4. **Outgoing data:** to build the response, the view passes the model instance(s) back through the serializer, which turns them into JSON.
5. The view returns that JSON as the HTTP response.

So the model owns *what the data is and how it's stored*, and the serializer owns *what shape it takes when it crosses the HTTP boundary* — keeping the view code itself mostly about "what HTTP verb happened" rather than manual JSON parsing.

## 7. One model and what it represents

A Django model is a Python class that defines a database table: each class attribute (a `models.CharField`, `models.OneToOneField`, etc.) becomes a column, and each instance of the class corresponds to a row. Django's ORM uses the model to generate the actual SQL, so day-to-day code reads/writes plain Python objects instead of hand-written SQL queries.

`NssUser` (`LearningAPI/models/people/nssuser.py`) represents a student or instructor at NSS (the school running this platform) — specifically, the NSS-specific information about a person that doesn't belong on Django's own built-in `User` model. It has a `OneToOneField` to `settings.AUTH_USER_MODEL`, meaning every `NssUser` extends exactly one Django `User` (which already handles login credentials, name, email) by adding fields Django doesn't know about: `slack_handle` and `github_handle`.

The API needs to track this because the platform's whole purpose is to manage a person's *learning journey*, not just authenticate them. Beyond the two stored fields, the class defines computed properties — `score` (a computed learning score based on achieved objectives and core-skill levels), `assessment_overview` (a list of the student's assessments and their review status), and `current_cohort` (which class/cohort they're currently in, with course/schedule info) — all of which only make sense in the context of "this is a student at a coding school," not a generic authenticated user. Without a model like this, there'd be nowhere to attach that NSS-specific, learning-related data to a person.

## 8. Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View | `notify()` — a function-based view decorated with `@api_view(['POST'])` in `LearningAPI/views/notify.py` (note: this project doesn't use class-based `APIView` anywhere; its plain views are all function-based, so there's no "class name" here — the equivalent is the decorated function itself) | Good for a single, one-off action that doesn't correspond to CRUD on a resource — `notify()` just sends a Slack message; there's no "list/retrieve/update/delete a notification" concept, so a plain view mapped to one URL is simpler than forcing it into a resource shape. |
| ViewSet | `CohortViewSet` — `LearningAPI/views/cohort_view.py` | Good when you have a model/resource (here, `Cohort`) that needs the standard set of operations — list, retrieve, create, update, destroy — plus maybe a few extra custom actions (via `@action`). A `ViewSet` bundles all of that into one class, and a DRF router auto-generates the matching URLs (`/cohorts/`, `/cohorts/<pk>/`, etc.), so you don't hand-wire a URL per operation. |

## 9. What replaces templates and why?

In classic Django MTV, the template is the presentation layer — it takes data the view hands it and renders it into an HTML page a browser displays to a human.

This project has no app-level HTML templates (the only `templates/` content is Django admin's own) because `learn-ops-api` isn't rendering pages for humans at all — it's a REST API meant to be consumed by a separate frontend client (e.g. a React app), over HTTP, as JSON. The role of "define the shape of what gets sent back" is instead played by the **serializer**: where a template says "put this data into this HTML structure," a serializer says "put this data into this JSON structure." The view still does the same job it always did (handle the request, talk to the model), but its output format is JSON built by a serializer instead of HTML built by a template.

This makes sense for a REST API because the API's job is to be a data layer other applications talk to, not to present a UI itself — presentation (what a user actually sees and clicks) is the frontend's responsibility, decoupled from this backend entirely. That separation is also *why* it's an API in the first place: the same JSON endpoints could be consumed by a web frontend, a mobile app, or another service, none of which would want server-rendered HTML anyway.