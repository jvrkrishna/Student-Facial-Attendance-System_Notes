# Day 4 — Student Login

Today we'll build only the login functionality.

The flow will be:

```text
Student
   ↓
Login
   ↓
Check username/password
   ↓
Check approval
   ↓
Approved → Dashboard
Pending  → Cannot login
```

We will keep the GUI basic and use your existing project-level template structure:

```text
student_attendance/
└── templates/
    ├── base.html
    └── accounts/
        ├── signup.html
        ├── signup_success.html
        └── login.html
```

## 1. Models

No model changes are required today.

Your existing User model already has:

```text
username
password
role
is_approved
face_image
```

So don't modify `accounts/models.py`.

## 2. Forms

Create the login form.

File path: `accounts/forms.py`

Add this below your existing `StudentSignupForm`:

```python
from django import forms
class LoginForm(forms.Form):
    username = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)
```

That's all we need.

## 3. Views

We need to authenticate the user and then check approval.

File path: `accounts/views.py`

Add this import:

```python
from django.contrib.auth import authenticate, login, logout
```

Your existing imports should remain.

Then add the login view:

File path: `accounts/views.py`

```python
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.forms import AuthenticationForm


def user_login(request):
    form = AuthenticationForm(request, data=request.POST or None)

    if request.method == "POST" and form.is_valid():
        user = form.get_user()

        if not user.is_approved:
            form.add_error(None, "Your account is waiting for approval.")
        else:
            login(request, user)
            return redirect("home")

    return render(request, "accounts/login.html", {"form": form})
```

We also need `LoginForm`.

File path: `accounts/views.py`

Change the existing forms import:

```python
from .forms import StudentSignupForm, LoginForm
```

## 4. URLs

File path: `accounts/urls.py`

Add:

```python
path("login/", views.user_login, name="login"),
```

## 5. Login GUI

Now create the login page.

File path: `student_attendance/templates/accounts/login.html`

```html
{% extends "base.html" %}

{% block title %}Login{% endblock %}

{% block content %}

<h2>Login</h2>

{% if error %}
    <p>{{ error }}</p>
{% endif %}

<form method="post">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">Login</button>

</form>

{% endblock %}
```

## 6. Test the GUI First

Start the server.

### Terminal command

```bash
python manage.py runserver
```

Open:

```text
http://127.0.0.1:8000/login/
```

You should see:

```text
Login

Username
[____________]

Password
[____________]

[ Login ]
```

## 7. Test Pending Student

Use the student account you created on Day 3.

Because:

```text
is_approved = False
```

the student should not be logged in.

Instead:

```text
Your account is waiting for approval.
```

This is important because your requirement says:

```text
Only approved students can log in.
```

## 8. Approve Student Temporarily

For testing Day 4, open Django Admin.

```text
http://127.0.0.1:8000/admin/
```

Go to:

```text
Accounts → Users → Your student
```

Change:

```text
Is approved
```

to:

```text
Yes
```

Save.

## 9. Test Approved Student

Return to:

```text
http://127.0.0.1:8000/login/
```

Enter the student's username and password.

Now Django should authenticate the student.

The view contains:

```text
authenticate()
      ↓
approved?
      ↓
login()
```

## 10. One Temporary Issue

The successful login currently redirects to:

```python
redirect("home")
```

If your project doesn't have a home URL yet, Django will give:

```text
NoReverseMatch
```

That's okay.

We haven't built the student dashboard yet.

For today's test, if you get this error after successful authentication, it means login itself worked; the destination page simply doesn't exist yet.

Don't add a dashboard today just to fix it.
