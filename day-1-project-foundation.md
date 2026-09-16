# Student Facial Attendance System

## Day 1 — Project Foundation

## Today's Goal

Today we will create only the basic Django project foundation:

- Create project folder
- Create virtual environment
- Install required packages
- Create Django project
- Create separate apps
- Configure MySQL
- Use .env only for DB_PASSWORD
- Create .gitignore
- Create project-level templates folder
- Create base.html
- Configure Django to find base.html
- Run migrations
- Verify the project

We will not create users, registration, login, facial recognition, location validation, or attendance today.

## 1. Create the project folder

### Terminal command

```bash
mkdir student_attendance
cd student_attendance
```

## 2. Create the virtual environment

### Terminal command

```bash
python -m venv venv
```

Activate it.

### Windows

```bash
venv\Scripts\activate
```

### Linux/macOS

```bash
source venv/bin/activate
```

Your terminal should now show something similar to:

```text
(venv)
```

## 3. Install required packages

For Day 1, install only the packages needed for the project foundation.

### Terminal command

```bash
pip install django mysqlclient python-dotenv
```

### Why these packages?

| Package | Purpose |
|---|---|
| Django | Web framework |
| mysqlclient | MySQL database connection |
| python-dotenv | Read DB_PASSWORD from .env |

We will install facial-recognition packages later when we reach that part of the project.

# Create requirements.txt

Save the installed packages.

### Terminal command

```bash
pip freeze > requirements.txt
```

This file will later be updated when we add facial-recognition and other dependencies.

## 4. Create the Django project

We will call the project configuration package `config`.

### Terminal command

```bash
django-admin startproject config .
```

The `.` means Django creates the project inside the current folder.

Initial structure:

```text
student_attendance/
│
├── manage.py
│
└── config/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

## 5. Create the separate applications

We will separate functionality into different apps.

### Terminal command

```bash
python manage.py startapp accounts
python manage.py startapp attendance
```

Add the four apps to `INSTALLED_APPS`.

File path: `config/settings.py`

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "accounts",
    "attendance",
]
```

## 6. Create the MySQL database

Make sure your local MySQL server is running.

Open MySQL.

### Terminal command

```bash
mysql -u root -p
```

Enter your MySQL password.

Create the database.

### MySQL command

```sql
CREATE DATABASE student_attendance CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### MySQL command

```sql
SHOW DATABASES;
```

You should see:

```text
student_attendance
```

Exit MySQL.

```sql
EXIT;
```

## 7. Configure MySQL in Django

Django's default database is SQLite. We need to replace it with MySQL.

Replace the existing `DATABASES` configuration.

File path: `config/settings.py`

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.mysql",
        "NAME": "student_attendance",
        "USER": "root",
        "PASSWORD": os.getenv("DB_PASSWORD"),
        "HOST": "127.0.0.1",
        "PORT": "3306",
    }
}
```

Notice the important point:

```python
"PASSWORD": os.getenv("DB_PASSWORD"),
```

The password comes from `.env`.

The other values remain directly in `settings.py`:

```text
NAME     → student_attendance
USER     → root
HOST     → 127.0.0.1
PORT     → 3306
```

So our configuration is intentionally:

```text
.env
└── DB_PASSWORD
```

## 8. Create .env

For this project, we will not put everything inside .env.

Only the database password will be stored there for now.

File path: `.env`

```env
DB_PASSWORD=your_mysql_password
```

Replace:

```text
your_mysql_password
```

with your actual MySQL password.

For example:

```env
DB_PASSWORD=MyPassword123
```

### Configure .env loading

We only need to load the `.env` file so Django can read `DB_PASSWORD`.

First, add the required imports.

File path: `config/settings.py`

Add below the existing:

```python
BASE_DIR = Path(__file__).resolve().parent.parent
```

this code:

```python
from dotenv import load_dotenv
load_dotenv(BASE_DIR / ".env")
```

## 9. Create .gitignore

The `.env` file contains a password, so it must not be uploaded to GitHub.

File path: `.gitignore`

```gitignore
venv/
**/__pycache__/
*.sqlite3
.env
media/
staticfiles/
```

The most important line today is:

```text
.env
```

## 10. Create the project-level templates folder

We want a single project-level `base.html`.

### Terminal command

```bash
mkdir templates
```

Our structure will be:

```text
student_attendance/
└── templates/
    └── base.html
```

### Configure the templates directory

File path: `config/settings.py`

```python
"DIRS": [BASE_DIR / "templates"],
```


### Create base.html

This is our project-level base template.

There will be no Bootstrap, CSS, JavaScript, or styling.

File path: `templates/base.html`

```html
<html>>
<head>
    <title>{% block title %}Student Attendance System{% endblock %}</title>
</head>
<body>
    <header>
        <h1>Student Facial Attendance System</h1>
        <nav>
            {% block navigation %}
            {% endblock %}
        </nav>
    </header>
    <main>
        {% block content %}
        {% endblock %}
    </main>
</body>
</html>
```

We are creating the basic structure now.

The actual Login, Signup, and User navigation will be added when we build the authentication functionality.



## 11. Check the Django project

### Terminal command

```bash
python manage.py check
```

Expected result:

```text
System check identified no issues (0 silenced).
```

If there is an error, stop here and fix it before continuing.

## 12. Run Django migrations

The built-in Django applications already have database models.

Run:

### Terminal command

```bash
python manage.py migrate
```

Django should create the required tables in your MySQL database.

## 13. Verify the MySQL tables

You can verify the database directly.

Open MySQL.

### Terminal command

```bash
mysql -u root -p
```

Select the project database.

### MySQL command

```sql
USE student_attendance;
```

Show the tables.

### MySQL command

```sql
SHOW TABLES;
```

You should see Django tables such as:

```text
auth_group
auth_permission
django_admin_log
django_content_type
django_migrations
django_session
...
```

This confirms that Django is using MySQL.

Exit:

### MySQL command

```sql
EXIT;
```

## 14. Start the Django development server

### Terminal command

```bash
python manage.py runserver
```

You should see something similar to:

```text
Starting development server at http://127.0.0.1:8000/
```

Open in your browser:

```text
http://127.0.0.1:8000/
```

For now, seeing Django's default page is fine.

We have not created the application homepage yet.

## 15. Day 1 Project Structure

After completing Day 1, your project should approximately look like this:

```text
student_attendance/
│
├── manage.py
├── .env
├── .gitignore
├── requirements.txt
│
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
├── accounts/
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
│
├── attendance/
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
└── templates/
    └── base.html
```
