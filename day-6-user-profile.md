# Day 6 — User Profile

Today we will build the User page that appears in the header after login.

The goal is small:

```text
Login
  ↓
Home
  ↓
User
  ↓
Profile information
```

We will not start attendance, facial recognition today.

We also won't create a new model because the information we need already exists in User.

## 1. Models

No changes.

File path: `accounts/models.py`

Do not modify this file today.

Your existing User model already contains:

```text
username
first_name
last_name
email
role
is_approved
face_image
```

## 2. Forms

No forms are required today.

We are only displaying the user's existing information.

File path: `accounts/forms.py`

Do not modify this file.

## 3. Views

We need one view for the User page.

File path: `accounts/views.py`

Add this view:

```python
@login_required
def user_profile(request):
    return render(request, "accounts/user_profile.html")
```

We need `login_required`.

File path: `accounts/views.py`

Add this import with your existing imports:

```python
from django.contrib.auth.decorators import login_required
```

If you already have it, don't add it again.

### Why login_required?

The User page should only be accessible after login.

If someone who is not logged in tries to open it, Django sends them to the login page.

## 4. URLs

Now connect the User page.

File path: `accounts/urls.py`

Add:

```python
path("user/", views.user_profile, name="user_profile"),
```

## 5. Connect the User Link

Previously, our header had in:

File path: `student_attendance/templates/base.html`

Find:

```html
<a href="#">User</a>
```

Replace only that line with:

```html
<a href="{% url 'user_profile' %}">User</a>
```

That's the only header change required.

## 6. Create User Profile GUI

Now create the page.

Remember your agreed template structure:

```text
student_attendance/
└── templates/
    ├── base.html
    └── accounts/
        ├── signup.html
        ├── signup_success.html
        ├── login.html
        ├── home.html
        └── user_profile.html
```

File path: `student_attendance/templates/accounts/user_profile.html`

```html
{% extends "base.html" %}

{% block title %}User Profile{% endblock %}

{% block content %}

<h2>User Profile</h2>

<p>Username: {{ user.username }}</p>

<p>First Name: {{ user.first_name }}</p>

<p>Last Name: {{ user.last_name }}</p>

<p>Email: {{ user.email }}</p>

<p>Role: {{ user.get_role_display }}</p>

<p>Approved: {{ user.is_approved }}</p>

{% if user.role == "STUDENT" %}
    <p>Face image: {{ user.face_image }}</p>
{% endif %}

{% endblock %}
```

This is intentionally basic.

## 7. Test the GUI

Start the server if it isn't running.

### Terminal command

```bash
python manage.py runserver
```

Login using your approved student.

Then click:

```text
User
```

You should see something similar to:

```text
User Profile

Username: student1

First Name: John

Last Name: Smith

Email: student1@example.com

Role: Student

Approved: True

Face image: student_faces/john.jpg
```

## 8. Test Direct Access Without Login

Logout.

Then manually open:

```text
http://127.0.0.1:8000/user/
```

You should be redirected to the login page.

This confirms:

```text
/user/
   ↓
@login_required
   ↓
Not logged in?
   ↓
Login page
```

## 9. Test Admin User

Log in with your Admin account.

Open:

```text
http://127.0.0.1:8000/user/
```

You should see:

```text
User Profile

Username: admin

Role: Admin

Approved: True
```

The face image should not be displayed because the user isn't a student.

## 10. Test Superadmin

Log in with your Superadmin account.

Open:

```text
http://127.0.0.1:8000/user/
```

You should see:

```text
User Profile

Username: superadmin

Role: Superadmin

Approved: True
```

Again, no face image is needed for Superadmin.

## 11. What We Have Now

Our basic account flow is:

```text
             Signup
                ↓
        Student pending
                ↓
       Superadmin approval
                ↓
              Login
                ↓
             Home
             /   \
            /     \
         User    Logout
```

The User page is currently read-only.

We are not adding profile editing yet because it isn't required for the attendance system at this stage.
