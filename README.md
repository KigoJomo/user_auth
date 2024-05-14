# User Registration and Authentication

## Basic procedure

### 1. Set Up the Django Project

1. **Install Django:**
   If you haven't installed Django yet, you can do so using pip:

   ```bash
   pip install django
   ```

2. **Create a new Django project:**

   ```bash
   django-admin startproject user_auth
   cd user_auth
   ```

3. **Create a new app within the project:**

   ```bash
   python manage.py startapp accounts
   ```

4. **Add the new app to your project settings:**
   Open `user_auth/settings.py` and add `'accounts'` to the `INSTALLED_APPS` list.

### 2. Configure Custom User Model

1. **Create a custom user model in the `accounts` app:**
   Open `accounts/models.py` and define your custom user model:

   ```python
   from django.contrib.auth.models import AbstractBaseUser, BaseUserManager
   from django.db import models

   class CustomUserManager(BaseUserManager):
       def create_user(self, email, password=None, **extra_fields):
           if not email:
               raise ValueError('The Email field must be set')
           email = self.normalize_email(email)
           user = self.model(email=email, **extra_fields)
           user.set_password(password)
           user.save(using=self._db)
           return user

       def create_superuser(self, email, password=None, **extra_fields):
           extra_fields.setdefault('is_staff', True)
           extra_fields.setdefault('is_superuser', True)

           return self.create_user(email, password, **extra_fields)

   class CustomUser(AbstractBaseUser):
       email = models.EmailField(unique=True)
       first_name = models.CharField(max_length=30)
       last_name = models.CharField(max_length=30)
       is_active = models.BooleanField(default=True)
       is_staff = models.BooleanField(default=False)
       is_superuser = models.BooleanField(default=False)

       objects = CustomUserManager()

       USERNAME_FIELD = 'email'
       REQUIRED_FIELDS = []

       def __str__(self):
           return self.email
   ```

2. **Update settings to use the custom user model:**
   In `user_auth/settings.py`, add:

   ```python
   AUTH_USER_MODEL = 'accounts.CustomUser'
   ```

### 3. Create User Registration Form

1. **Create a form for user registration:**
   Create a new file `accounts/forms.py` and add the following:

   ```python
   from django import forms
   from django.contrib.auth.forms import UserCreationForm
   from .models import CustomUser

   class CustomUserCreationForm(UserCreationForm):
       class Meta:
           model = CustomUser
           fields = ('email', 'first_name', 'last_name')
   ```

### 4. Set Up Views and URLs

1. **Create views for registration and login:**
   Open `accounts/views.py` and add:

   ```python
   from django.shortcuts import render, redirect
   from django.contrib.auth import login, authenticate
   from .forms import CustomUserCreationForm

   def register(request):
       if request.method == 'POST':
           form = CustomUserCreationForm(request.POST)
           if form.is_valid():
               user = form.save()
               login(request, user)
               return redirect('home')
       else:
           form = CustomUserCreationForm()
       return render(request, 'accounts/register.html', {'form': form})

   def home(request):
       return render(request, 'accounts/home.html')
   ```

2. **Create templates for registration and home:**
   Create a directory `templates/accounts/` and add `register.html` and `home.html`.

   `register.html`:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Register</title>
   </head>
   <body>
       <h2>Register</h2>
       <form method="post">
           {% csrf_token %}
           {{ form.as_p }}
           <button type="submit">Register</button>
       </form>
   </body>
   </html>
   ```

   `home.html`:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Home</title>
   </head>
   <body>
       <h2>Home</h2>
       <p>Welcome, {{ user.email }}!</p>
   </body>
   </html>
   ```

3. **Configure URLs:**
   Create `accounts/urls.py` and define the URL patterns:

   ```python
   from django.urls import path
   from .views import register, home

   urlpatterns = [
       path('register/', register, name='register'),
       path('', home, name='home'),
   ]
   ```

   Include these URLs in the project's `urls.py`:

   ```python
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('accounts/', include('accounts.urls')),
   ]
   ```

### 5. Configure Authentication Backend

1. **Update the authentication backend:**
   In `user_auth/settings.py`, ensure Django uses the email as the authentication field:

   ```python
   AUTHENTICATION_BACKENDS = [
       'django.contrib.auth.backends.ModelBackend',
   ]
   ```

2. **Create login form and view:**
   Add a login form to `accounts/forms.py`:

   ```python
   from django.contrib.auth.forms import AuthenticationForm

   class CustomAuthenticationForm(AuthenticationForm):
       username = forms.EmailField(label='Email')
   ```

   Create a login view in `accounts/views.py`:

   ```python
   from django.contrib.auth.views import LoginView

   class CustomLoginView(LoginView):
       authentication_form = CustomAuthenticationForm
       template_name = 'accounts/login.html'
   ```

   Add a URL pattern for login in `accounts/urls.py`:

   ```python
   from django.urls import path
   from django.contrib.auth import views as auth_views
   from .views import register, home, CustomLoginView

   urlpatterns = [
       path('register/', register, name='register'),
       path('login/', CustomLoginView.as_view(), name='login'),
       path('', home, name='home'),
   ]
   ```

   Create `login.html` in `templates/accounts/`:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Login</title>
   </head>
   <body>
       <h2>Login</h2>
       <form method="post">
           {% csrf_token %}
           {{ form.as_p }}
           <button type="submit">Login</button>
       </form>
   </body>
   </html>
   ```

### 6. Apply Migrations and Test

1. **Apply migrations:**

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Create a superuser for testing:**

   ```bash
   python manage.py createsuperuser
   ```

3. **Run the development server:**

   ```bash
   python manage.py runserver
   ```

4. **Test the registration and login functionalities:**
   - Navigate to `http://127.0.0.1:8000/accounts/register/` to register a new user.
   - Navigate to `http://127.0.0.1:8000/accounts/login/` to log in with an existing user.

This setup provides a basic implementation of user registration and login using email instead of a username in Django. For production use, ensure you add more features like email verification, password reset, and enhanced security practices.
