# Day 3 — Student Signup + Face Image

## Today's Goal

Build:

```text
Student
   ↓
Signup GUI
   ↓
Personal details
   ↓
Face image
   ↓
Account created
   ↓
Pending approval
```

The uploaded face image will later be used as the student's reference image for daily facial verification.

Today we will not build facial recognition itself.

## 1. Forms

Now create the student signup form.

File path: `accounts/forms.py`

Create:

```python
from django.contrib.auth.forms import UserCreationForm
from .models import User


class StudentSignupForm(UserCreationForm):
    class Meta:
        model = User
        fields = [
            "username",
            "first_name",
            "last_name",
            "email",
            "face_image",
        ]
```

# or

```python
from django import forms
from .models import User

class StudentSignupForm(forms.ModelForm):
    class Meta:
        model = User
        fields = [
            "username",
            "first_name",
            "last_name",
            "email",
            "face_image",
        ]
        
    password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)

    def clean(self):
        cleaned_data = super().clean()

        password = cleaned_data.get("password")
        confirm_password = cleaned_data.get("confirm_password")

        if password and confirm_password and password != confirm_password:
            raise forms.ValidationError("Passwords do not match.")

        return cleaned_data
```

### Important

The student does not select:

```text
Role

Is approved
```

Those values are controlled by the server.

The face image is required because:

```python
face_image = forms.ImageField()
```

## 3. Views

Now connect the form to the signup page.

File path: `accounts/views.py`

Replace the existing contents with:
```python
from django.shortcuts import render, redirect
from .forms import StudentSignupForm

def student_signup(request):
    if request.method == "POST":
        form = StudentSignupForm(request.POST, request.FILES)

        if form.is_valid():
            form.save()
            return redirect("signup_success")

    else:
        form = StudentSignupForm()

    return render(request, "accounts/signup.html", {"form": form})

def signup_success(request):
    return render(request, "accounts/signup_success.html")

```
or 

```python
from django.shortcuts import render, redirect
from .forms import StudentSignupForm

def student_signup(request):
    if request.method == "POST":
        form = StudentSignupForm(request.POST, request.FILES)

        if form.is_valid():
            user = form.save(commit=False)
            user.set_password(form.cleaned_data["password"])
            user.save()

            return redirect("signup_success")

    else:
        form = StudentSignupForm()
    return render(request, "accounts/signup.html", {"form": form})


def signup_success(request):
    return render(request, "accounts/signup_success.html")
```


## 4. URLs

File path: `accounts/urls.py`

Create this if it doesn't already exist:

```python
from django.urls import path

from . import views


urlpatterns = [
    path("signup/", views.student_signup, name="signup"),
    path("signup/success/", views.signup_success, name="signup_success"),
]
```

## 5. Connect App URLs

File path: `config/urls.py`

Make sure this import exists:

```python
from django.urls import include, path
path("", include("accounts.urls")),
```


## 6. Templates

Now we create the GUI.

File path: `templates/accounts/signup.html`

```html
{% extends "base.html" %}

{% block title %}Student Signup{% endblock %}

{% block content %}

<h2>Student Signup</h2>

<form method="post" enctype="multipart/form-data">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">Sign Up</button>

</form>

{% endblock %}
```

The important part is:

```html
enctype="multipart/form-data"
```

This allows the browser to send the image to Django.

## 7. Success Page

File path: `templates/accounts/signup_success.html`

```html
{% extends "base.html" %}

{% block title %}Signup Successful{% endblock %}

{% block content %}

<h2>Registration Successful</h2>

<p>Your student account has been created.</p>

<p>Your account is waiting for superadmin approval.</p>

<p>You can log in after your account has been approved.</p>

{% endblock %}
```

## 8. Media Configuration

The uploaded face image needs to be stored locally.

File path: `config/settings.py`

Add:

```python
MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"
```

## 9. Media URL

File path: `config/urls.py`

Add these imports:

```python
from django.conf import settings
from django.conf.urls.static import static
```

Add:

```python
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

This is for local development only.

## 10. Check Django

### Terminal command

```bash
python manage.py check
```

Expected:

```text
System check identified no issues (0 silenced).
```

## 11. Run the Server

### Terminal command

```bash
python manage.py runserver
```

Open:

```text
http://127.0.0.1:8000/signup/
```

## 12. First Look at the GUI

You should see something like:

```text
Student Signup

Username

[______________]

First name

[______________]

Last name

[______________]

Email

[______________]

Password

[______________]

Confirm password

[______________]

Face image

[Choose File]

[ Sign Up ]
```

This is intentionally basic.

No:

- Bootstrap
- CSS
- JavaScript
- fancy UI

## 13. Test Without Image

Try submitting the form without selecting an image.

It should fail because the face image is required.

This confirms:

```python
face_image = forms.ImageField()
```

is working.

## 14. Test With Image

Choose a clear face image.

Enter something like:

```text
Username: student2
First name: John
Last name: Smith
Email: student2@example.com
Password: ********
Confirm password: ********
Face image: john.jpg
```

Click:

```text
Sign Up
```

You should see the registration success page.

## 15. Check Django Admin

Open:

```text
http://127.0.0.1:8000/admin/
```

Go to:

```text
Accounts → Users
```

The new student should show:

```text
Role: Student

Is approved: No

Face image: student_faces/...
```

So the result is:

```text
student2

│

├── Role: Student

├── Is approved: No

└── Face image
```

## 16. Why the Image Is Collected Now

This image becomes the student's reference face.

Later:

```text
Signup
   ↓
Face image
   ↓
Pending approval
   ↓
Superadmin approves
   ↓
Student logs in
   ↓
Daily attendance
   ↓
Camera captures current face
   ↓
Compare with registered face
   ↓
Verification result
```

We don't need facial-recognition code today.

## 17. Camera — Later

For now:

```text
Signup → Upload Image
```

Later we can make it:

```text
Signup

   ↓

Upload Image OR Camera

          ↓

       Capture

          ↓

      face_image
```

Camera access will require browser permission, and deployment will normally require HTTPS.

We'll handle that when we reach the camera/facial-recognition stage.
