# Day 5 — Login Success + User Header

Today we will make the login flow visible and useful.

After Day 4, the student can authenticate, but we don't yet have a proper page to land on. Today we'll create a simple home/dashboard page and update the header according to login status.

We will keep it very small.

## Today's flow

```text
Signup
   ↓
Pending
   ↓
Superadmin approves
   ↓
Login
   ↓
Student Home
```

And the header:

```text
Not logged in:
Home | Login | Signup

Logged in:
Home | User | Logout
```

No attendance, camera, location, or facial recognition yet.

## 1. Models

No model changes today.

Don't modify:

```text
accounts/models.py
```

## 2. Forms

No form changes today.

Don't modify:

```text
accounts/forms.py
```

## 3. Views

We need two things:

- A home page.
- Logout.

We'll keep these in the accounts app for now because they are basic user/account functionality.

File path: `accounts/views.py`

Add this import:

```python
from django.contrib.auth import logout
```

If you already have this import from Day 4, don't add it again.

Then add these views at the bottom:

File path: `accounts/views.py`

```python
def home(request):
    return render(request, "accounts/home.html")


def user_logout(request):
    logout(request)
    return redirect("home")
```

That's all.

## 4. URLs

File path: `accounts/urls.py`

Add these two paths to your existing `urlpatterns`:

```python
path("", views.home, name="home"),
path("logout/", views.user_logout, name="logout"),
```

Your existing signup and login URLs remain unchanged.

The important URLs are now:

```text
/              → Home
/signup/       → Signup
/login/        → Login
/logout/       → Logout
```

## 5. Create the Home GUI

Now we'll create the page the user sees after login.

Remember your template structure:

```text
student_attendance/
└── templates/
    ├── base.html
    └── accounts/
```

File path: `student_attendance/templates/accounts/home.html`

```html
{% extends "base.html" %}

{% block title %}Home{% endblock %}

{% block content %}

{% if user.is_authenticated %}

    <h2>Welcome, {{ user.username }}</h2>

    <p>Role: {{ user.get_role_display }}</p>

    {% if user.role == "STUDENT" %}
        <p>Student account</p>
    {% endif %}

{% else %}

    <h2>Welcome to Attendance System</h2>

    <p>Please login or signup.</p>

{% endif %}

{% endblock %}
```

This is intentionally basic.

## 6. Update the Base Header

Now we make the header change according to the user's status.

You already have:

```text
student_attendance/templates/base.html
```

We only need to modify the navigation/header area.

File path: `student_attendance/templates/base.html`

Find your current header/navigation links.

Replace only the navigation part with:

```html
<nav>

    <a href="{% url 'home' %}">Home</a>

    {% if user.is_authenticated %}

        <a href="{% url 'logout' %}">Logout</a>

        <a href="#">User</a>

    {% else %}

        <a href="{% url 'login' %}">Login</a>
        <a href="{% url 'signup' %}">Signup</a>

    {% endif %}

</nav>
```

### Result

Before login:

```text
Attendance System

Home | Login | Signup
```

After login:

```text
Attendance System

Home | Logout | User
```

We are using a temporary `#` for User because we haven't built the user page yet.

We will connect that link when we actually create the user page.

## 7. Test the GUI First

Start the server.

### Terminal command

```bash
python manage.py runserver
```

Open:

```text
http://127.0.0.1:8000/
```

You should see:

```text
Attendance System

Home | Login | Signup

Welcome to Attendance System

Please login or signup.
```

## 8. Test Login

Before testing, make sure your student has:

```text
Role: Student
Is approved: Yes
```

You can temporarily approve the student from Django Admin.

Then open:

```text
http://127.0.0.1:8000/login/
```

Enter the student's username and password.

After successful login, you should reach:

```text
Welcome, student2

Role: Student

Student account
```

The header should now change to:

```text
Home | Logout | User
```

## 9. Test Logout

Click:

```text
Logout
```

You should return to:

```text
Home
```

The header should change back to:

```text
Home | Login | Signup
```

This confirms the Django session was removed.

## 10. Test Pending Student Again

This is important.

In Django Admin:

```text
Accounts → Users → Student
```

Set:

```text
Is approved = No
```

Then logout and try logging in.

The student should receive:

```text
Your account is waiting for approval.
```

The student should not enter the authenticated area.

## 11. What We Have Now

The basic user flow is now:

```text
                 ┌──────────────┐
                 │    Signup    │
                 └──────┬───────┘
                        ↓
                 Pending student
                        ↓
                 Superadmin approves
                        ↓
                     Login
                        ↓
                  Student Home
                        ↓
                  User / Logout
```

And the header responds to authentication:

```text
Not logged in
     ↓
Login | Signup
```

```text
Logged in
     ↓
User | Logout
```

## 12. Why We Stop Here

We are not adding attendance yet.

The next major feature needs its own app and workflow:

```text
attendance
```

Before that, we need to establish the student's basic authenticated area.

Later the student area will become something like:

```text
Student Dashboard

Welcome, student1

[ Submit Attendance ]

[ My Attendance ]

Face verification status
Location status
```

But we won't build those buttons until their actual functionality is ready.
