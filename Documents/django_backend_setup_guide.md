# Django Backend Setup — Intelligent Freight Quote Generation System

A complete step-by-step runbook: system installs, VS Code setup, and the full Django project scaffold matching the architecture in the main documentation.

---

## Phase 0 — What You Need Installed

| Tool | Why | Check if already installed |
|---|---|---|
| Python 3.11 or 3.12 | Runs Django | `python --version` |
| pip | Installs Python packages | `pip --version` |
| Git | Version control (Section 16 of the doc) | `git --version` |
| VS Code | Editor | Open the app |
| MongoDB (local or Atlas) | Database | — |

Run the three check commands first in a terminal. If any fail with "command not found," install that tool using the steps below.

---

## Phase 1 — Install Python

**Windows:**
1. Go to python.org/downloads and get the latest Python 3.12 installer.
2. Run it. **Check the box "Add python.exe to PATH"** before clicking Install — this is the step people most often miss.
3. Open a new Command Prompt and confirm:
```bash
python --version
pip --version
```

**Mac:**
```bash
brew install python@3.12
python3 --version
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
python3 --version
```

> If `python` doesn't work on Mac/Linux, use `python3` and `pip3` for every command below.

---

## Phase 2 — Install VS Code + Extensions

1. Download from code.visualstudio.com and install it.
2. Open VS Code → Extensions icon (left sidebar) → install these four:
   - **Python** (by Microsoft)
   - **Pylance** (by Microsoft)
   - **Django** (by Baptiste Darthenay) — adds syntax highlighting for templates
   - **Thunder Client** — lets you test your API endpoints without leaving VS Code (a lightweight Postman)
3. Also install **MongoDB for VS Code** if you want to browse your database visually.

---

## Phase 3 — Install Git & Configure It

**Windows:** download from git-scm.com, install with default options.
**Mac:** `brew install git`
**Linux:** `sudo apt install git`

Then set your identity once:
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## Phase 4 — Get MongoDB Running

Easiest option for tomorrow — **MongoDB Atlas (free cloud cluster)**, no local install needed:
1. Go to mongodb.com/cloud/atlas → sign up → create a free (M0) cluster.
2. Under **Database Access**, create a username/password.
3. Under **Network Access**, add IP `0.0.0.0/0` (allow from anywhere — fine for training/dev).
4. Click **Connect → Drivers → Python**, copy the connection string. It looks like:
```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/
```
Keep this string — you'll paste it into `.env` in Phase 7.

*(If you'd rather run MongoDB locally instead, say so and I'll give you the local install steps for your OS.)*

---

## Phase 5 — Create the Project Folder & Virtual Environment
Open a terminal where you want the project to live (e.g. next to your `frontend/` folder from today):

```bash
mkdir backend
cd backend
python -m venv venv
```
Activate the virtual environment:
```bash
# Windows
venv\Scripts\activate

# Mac / Linux
source venv/bin/activate
```
You'll know it worked because your terminal prompt now starts with `(venv)`. **Do this every time you open a new terminal to work on the backend.**

Open the `backend` folder in VS Code:
```bash
code .
```
Then in VS Code, press `Ctrl+Shift+P` (Windows) or `Cmd+Shift+P` (Mac) → type "Python: Select Interpreter" → choose the one inside `venv`.

---

## Phase 6 — Install Django & Required Packages

With `(venv)` active:

```bash
pip install django
pip install djangorestframework
pip install djangorestframework-simplejwt
pip install django-cors-headers
pip install pymongo
pip install python-decouple
```

| Package | Purpose |
|---|---|
| `django` | The core framework |
| `djangorestframework` | Builds the REST APIs (Section 10 of the doc) |
| `djangorestframework-simplejwt` | JWT authentication (Section 13) |
| `django-cors-headers` | Lets your React app (different port) call this API |
| `pymongo` | Direct MongoDB driver — simplest, most reliable way to talk to MongoDB from Django |
| `python-decouple` | Keeps secrets (Mongo URI, JWT key) out of your code, in a `.env` file |

Save the exact versions so anyone can reinstall the same setup later:
```bash
pip freeze > requirements.txt
```

---

## Phase 7 — Create the Django Project

```bash
django-admin startproject freight_project .
```
(the trailing `.` creates it in the current `backend` folder instead of nesting it one level deeper)

Create a `.env` file in `backend/` (same level as `manage.py`):
```
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/
MONGO_DB_NAME=freight_db
JWT_SECRET=change-this-to-something-random
DEBUG=True
```

---

## Phase 8 — Create the Apps (Modules)

Each of these matches a module from Section 7/8 of the main documentation:

```bash
python manage.py startapp authentication
python manage.py startapp shipment
python manage.py startapp pricing
python manage.py startapp ml_engine
python manage.py startapp history
python manage.py startapp admin_panel
python manage.py startapp reports
mkdir common
```

Your folder should now look like:
```text
backend/
├── venv/
├── freight_project/
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── authentication/
├── shipment/
├── pricing/
├── ml_engine/
├── history/
├── admin_panel/
├── reports/
├── common/
├── .env
├── manage.py
└── requirements.txt
```

---

## Phase 9 — Configure `settings.py`

Open `freight_project/settings.py` in VS Code and make these changes:

**1. Add apps** — find `INSTALLED_APPS` and add:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',
    'corsheaders',

    'authentication',
    'shipment',
    'pricing',
    'ml_engine',
    'history',
    'admin_panel',
    'reports',
]
```

**2. Add CORS middleware** — find `MIDDLEWARE` and add this as the **first** item:
```python
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # ...keep the rest as-is
]
```

**3. Allow your React dev server** — add near the bottom:
```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",   # Vite's default dev port
]
```

**4. Configure JWT as the default auth method** — add:
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}
```

**5. Load your `.env` values** — near the top of the file:
```python
from decouple import config

MONGO_URI = config('MONGO_URI')
MONGO_DB_NAME = config('MONGO_DB_NAME')
SECRET_KEY = config('JWT_SECRET')
```

---

## Phase 10 — Connect to MongoDB

Create `common/db.py`:
```python
from pymongo import MongoClient
from django.conf import settings

client = MongoClient(settings.MONGO_URI)
db = client[settings.MONGO_DB_NAME]
```

Any app can now import and use it, e.g. inside `shipment/views.py`:
```python
from common.db import db

shipments_collection = db["shipments"]
```

This is the simplest possible bridge between Django and MongoDB — no ORM translation layer to fight with, which matters most for a training project on a deadline.

---

## Phase 11 — Wire Up the Root URLs

Open `freight_project/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/auth/', include('authentication.urls')),
    path('api/v1/shipment/', include('shipment.urls')),
    path('api/v1/pricing/', include('pricing.urls')),
    path('api/v1/ml/', include('ml_engine.urls')),
    path('api/v1/history/', include('history.urls')),
    path('api/v1/admin/', include('admin_panel.urls')),
    path('api/v1/reports/', include('reports.urls')),
]
```

Each app needs its own `urls.py` — for example, `authentication/urls.py`:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('register', views.register, name='register'),
    path('login', views.login, name='login'),
]
```
(You'll add the matching `views.py` functions as you build each module out — start with `authentication` since everything else depends on it.)

---

## Phase 12 — Run It

```bash
python manage.py runserver
```

Open a browser to `http://127.0.0.1:8000/` — you should see Django's default welcome page. That confirms Python, Django, settings, and the app registrations are all working together.

Test with Thunder Client in VS Code by hitting one of your endpoints once you've written its view function.

---

## Phase 13 — Git Init (matches Section 16 of the main doc)

Create `.gitignore` in `backend/`:
```text
venv/
__pycache__/
*.pyc
.env
db.sqlite3
```

Then:
```bash
git init
git add .
git commit -m "feat: initial Django project setup with app scaffolding"
```

---

## Tomorrow's Build Order (Recommended)

1. `authentication` — register/login/JWT issuance first, since every other module needs a logged-in user.
2. `shipment` — the form-submission model and validation.
3. `pricing` — the rule-based engine (Section 11).
4. `ml_engine` — start with a stub that returns a fixed number, wire in the real model later.
5. `history`, `admin_panel`, `reports` — once the core quote flow works end-to-end.

---

## Quick Troubleshooting

| Problem | Fix |
|---|---|
| `python` not recognized | Re-run the Python installer, tick "Add to PATH" |
| `(venv)` not showing in prompt | Run the activate command again — you're in a new terminal |
| `ModuleNotFoundError: decouple` etc. | You forgot to activate `venv` before `pip install` |
| CORS error in browser console | Check `CORS_ALLOWED_ORIGINS` matches your Vite port exactly |
| MongoDB connection timeout | Check Atlas Network Access allows your IP (or `0.0.0.0/0`) |
