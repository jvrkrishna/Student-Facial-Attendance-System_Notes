# Day 2 — User Roles & Django Admin

## Today's Goal

By the end of Day 2, you should be able to open Django Admin and see:

- Users
- Student
- Admin
- Superadmin
- Student approval status by superadmin


## 1. First understand what we are building

We need three types of users:

| User | Role | Student approval |
|---|---|---|
| Superadmin | SUPERADMIN | Can approve students |
| Admin/Staff | ADMIN | Cannot approve students |
| Student | STUDENT | Must be approved by superadmin |

For these we are using AbstractUser that mean Django already provides:

```text
username
password
email
first name
last name
staff status
superuser status
permissions
```

So we don't need to recreate those. For now, we only need three additional fields of information on Django's:

- Role
- Approval status
- face_image (for facial recognisation. why we are creating means if we created later we will face some issue while doing migrations.)

if we want to use face_image field in model then you need to install

```python
pip install pillow
pip freeze>requirements.txt
```

So we will create a small custom User model.

## 2. Create the custom User model and add in settings.py

We already have the accounts app from Day 1.

We only need to modify its model.

File path: `accounts/models.py`

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    role = models.CharField(max_length=20,choices=[("STUDENT", "Student"),("ADMIN", "Admin"),("SUPERADMIN", "Superadmin"),],default="STUDENT",)
    is_approved = models.BooleanField(default=False)
    face_image = models.ImageField(upload_to="student_faces/",blank=False,null=False,)

    def __str__(self):
        return self.username
```

### What did we added?

Only these fields:

```text
role
is_approved
face_image
```

### Tell Django about our User

Django needs to know that accounts.User is now our user model.

File path: `config/settings.py`

Add:

```python
AUTH_USER_MODEL = "accounts.User"
```

## 3. Create the database, migration and migrate

### First create database in mysql

```sql
DROP DATABASE student_attendance;
CREATE DATABASE student_attendance;
```

```bash
python manage.py makemigrations accounts
```

You should see something similar to:

```text
Migrations for 'accounts':
  accounts/migrations/0001_initial.py
```

### Now convert our model into a database table.

```bash
python manage.py migrate
```

If this finishes without an error, our user model is now connected to MySQL.

### Now create super user again if already created also because just we deleted database

```bash
python manage.py createsuperuser
```

### If we need check in mysql

```sql
use student_attendance;
show tables;
describe accounts_user;
```

## 4. Show the User in Django Admin

Now we want to see our user model visually in admin panel.

File path: `accounts/admin.py`

Replace the existing contents with:

```python
from django.contrib import admin
from .models import User
admin.site.register(User)

#or

from django.contrib import admin
from .models import User
from django.contrib.auth.admin import UserAdmin
@admin.register(User)   #or admin.site.register(User, CustomUserAdmin)
class CustomUserAdmin(UserAdmin):
    fieldsets = UserAdmin.fieldsets + (("Attendance System",{"fields":("role","is_approved","face_image",)},),)
```

That's enough for now.

We don't need a custom admin interface or extra code.

## 5. Open the GUI

Start Django.

### Terminal command

```bash
python manage.py runserver
```

Open:

```text
http://127.0.0.1:8000/admin/
```

You should see the Django Admin login page.

Log in using the superuser you just created.

### Check the User GUI

Inside Django Admin, you should now see:

```text
Accounts

    Users
```

Click Users.

You should see your superuser.

Open that user.

You should see our custom section:

```text
Attendance System

Role
Is approved
```

This is the first important GUI result of Day 2.


## 6. Create a test Student

From the Django Admin GUI, create a new user.

Set:

```text
Username: student1
Role: Student
Is approved: No
```

Save the user.

Now we have:

```text
student1
   │
   ├── Role: Student
   └── Approved: No
   |__ Face_image
```

This represents a student waiting for approval.

## 7. Create a test Admin

Create another user from the GUI.

Set:

```text
Username: admin1
Role: Admin
Is approved: Yes
```

Do not make this user a superuser.

The important distinction is:

```text
Admin
is_staff = Yes
is_superuser = No
Face_image 
```

For now, Django Admin permissions can be managed separately. Later, we'll restrict what this role can actually do in our application.

## 8. Superadmin

Your original superuser should represent:

```text
Role: admin
Is approved: Yes
is_staff: Yes
is_superuser: Yes
Face_image
```

The three users now look conceptually like:

```text
                    Users
                      │
       ┌──────────────┼──────────────┐
       │              │              │
   Superadmin       Admin         Student
       │              │              │
     Full          Limited      Pending/
   permissions   Permissions    approved
```

