# Day 7 — Superadmin Student Approval

Today we will build the student approval page.

This is an important part of your requirement:

```text
Student signs up
       ↓
is_approved = False
       ↓
Superadmin sees pending students
       ↓
Superadmin approves
       ↓
is_approved = True
       ↓
Student can log in
```

An Admin/Staff user must not be able to approve students.

We will keep today's implementation small.

## 1. Today's GUI

First, the page we want:

```text
Student Approval

Pending Students

student1
Email: student1@example.com
Role: Student
Face image: student_faces/abc.jpg

[ Approve ]

student2
Email: student2@example.com
Role: Student
Face image: student_faces/xyz.jpg

[ Approve ]
```

Only a Superadmin can access this page.

No CSS or JavaScript.

## 2. Models

No model changes today.

Your existing User model already has:

```text
role
is_approved
face_image
```

So don't modify:

File path: `accounts/models.py`

Nothing to change.

## 3. Forms

We don't need a form for approval.

The approval action will simply be a POST request from an Approve button.

So don't modify:

File path: `accounts/forms.py`

## 4. Views

We need:

- A page showing pending students.
- An approval action.

File path: `accounts/views.py`

Add this view:

```python
from .models import User
@login_required
def student_approvals(request):
    if request.user.role != 'SUPERADMIN':
        return redirect("home")

    pending_students = User.objects.filter(role='STUDENT',is_approved=False,)

    return render(request,"accounts/student_approvals.html",{"pending_students": pending_students},)
    
```

This retrieves only:

```text
Student
AND
is_approved = False
```

## 5. Approval View

File path: `accounts/views.py`

Add:

```python
@login_required
def approve_student(request, user_id):
    if request.user.role != 'SUPERADMIN':
        return redirect("home")

    if request.method == "POST":
        student = User.objects.get(id=user_id,role='STUDENT',)

        student.is_approved = True
        student.save()
    return redirect("student_approvals")
```

So:

```text
Superadmin
   ↓
POST Approve
   ↓
is_approved = True
```

An Admin cannot perform this action because of:

```python
if request.user.role != User.Role.SUPERADMIN:
```

## 6. URLs

File path: `accounts/urls.py`

Add these two URLs:

```python
path("student-approvals/",views.student_approvals, name="student_approvals",),
    path("student-approvals/<int:user_id>/approve/",views.approve_student,name="approve_student",),
```

Your existing URLs remain unchanged.

## 7. Approval GUI

Now create the page in your agreed project-level template structure.

File path: `student_attendance/templates/accounts/student_approvals.html`

```html
{% extends "base.html" %}

{% block title %}Student Approvals{% endblock %}

{% block content %}

<h2>Student Approvals</h2>

{% if pending_students %}

    {% for student in pending_students %}

        <div>
            <h3>{{ student.username }}</h3>

            <p>Email: {{ student.email }}</p>

            <p>Role: {{ student.get_role_display }}</p>

            <p>Face image: {{ student.face_image }}</p>

            <form method="post"
                  action="{% url 'approve_student' student.id %}">

                {% csrf_token %}

                <button type="submit">Approve</button>

            </form>
        </div>

        <hr>

    {% endfor %}

{% else %}

    <p>No pending student registrations.</p>

{% endif %}

{% endblock %}
```

## 8. Add Approval Link to Header

We want the Superadmin to easily reach the approval page.

File path: `student_attendance/templates/base.html`

Inside the existing logged-in section means inside if user is authenticated, add:

```html
{% if user.role == "SUPERADMIN" %}
    <a href="{% url 'student_approvals' %}">Student Approvals</a>
{% endif %}
```

So a Superadmin will see:

```text
Home | Student Approvals | User | Logout
```

A normal Student will not see Student Approvals.

An Admin will also not see Student Approvals.

## 9. Test Superadmin

Log in as your Superadmin.

Open:

```text
http://127.0.0.1:8000/student-approvals/
```

You should see the pending students.

For example:

```text
Student Approvals

student1

Email: student1@example.com

Role: Student

Face image: student_faces/student1.jpg

[ Approve ]
```

## 10. Approve a Student

Click:

```text
Approve
```

The student should disappear from the pending list.

Check Django Admin:

```text
Accounts → Users → student1
```

You should now have:

```text
Role: Student
Is approved: Yes
Face image: student_faces/student1.jpg
```

## 11. Test Student Login

Logout from Superadmin.

Go to:

```text
http://127.0.0.1:8000/login/
```

Login as the approved student.

It should now work.

The flow is:

```text
Student signup
       ↓
is_approved = False
       ↓
Cannot login
       ↓
Superadmin approves
       ↓
is_approved = True
       ↓
Student can login
```

## 12. Test Admin Restriction

This is very important.

Log in using your Admin account.

Try to open:

```text
http://127.0.0.1:8000/student-approvals/
```

The Admin should be redirected to:

```text
Home
```

The Admin also should not see:

```text
Student Approvals
```

in the header.

Therefore:

```text
                  Student Approval
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
        Superadmin                 Admin
             ↓                       ↓
          Allowed                 Blocked
```

## 13. One Important Improvement

We are currently using:

```python
User.objects.get(...)
```

For this basic version, that's okay because the URL contains the student's ID.

Later, we can make the permission handling cleaner using Django's permission system.

Don't add that complexity now.

Your role-based requirement is already satisfied.

## 14. Current Project Flow

After Day 7, your system has:

```text
                 SIGNUP
                    │
                    ↓
              Student User
                    │
          is_approved = False
                    │
                    ↓
             Pending Student
                    │
                    ↓
              Superadmin
                    │
                    ↓
                APPROVE
                    │
                    ↓
          is_approved = True
                    │
                    ↓
              Student Login
```

And:

```text
Superadmin
 ├── Login
 ├── User
 ├── Logout
 └── Student Approvals

Admin
 ├── Login
 ├── User
 └── Logout

Student
 ├── Login
 ├── User
 └── Logout
```
