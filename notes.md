# Django

## Content
[Start](#Start), [Errors with Imports](#Imports),  [Environmental variable](#Env_variables),[Templates](#Templates), [Static files](#Static-files), [Models](#Models), [Population scripts using Faker]( #Population-script), [Databases](#Databases), [Forms](#Forms), [Relative URLs](#Relative-URLs), [Template & Custom Filters](#Template-filters), [Auth0 & AuthZ](#Auth0-AuthZ)

Django apps can be plugged into other apps and projects.

## Start:
- Initialize environment
    ```shell
    source activate <env_name>
    ```
- Create project
    ```shell
    django-admin startproject <Project_Name>
    ```
- Create an app within the project
    ```shell
    python manage.py startapp <app_name>
    ```
- Create a superuser for so as to use the admin interface and database
    ```shell
    # Ensure all settings are valid
    python manage.py createsuperuser
    ```
- Add `app_name` to the project in `project_name`'s `settings.py` file `INSTALLED_APPS` variable.
- Create views
- Map created view to a url in `Project_Name`'s `url.py`file
    - Import `views` into `url.py`
    ```python
    from <app_name> import views
    ```
    - Add to the `urlpatterns`variable:
    ```python
    path('<url>', views.<func_name>, name="<name>"),
    ```
- Alternative method for mapping url's is to create `url.py` files within each app, with the code above, then including them in the project's `url.py` file
    ```python
    #...
    from django.urls import include
    #...
    urlpatterns = [
        path('<path_name>', include('<app_name>.urls')),
        #...
    ```
- To run project
    ```shell
    python manage.py runserver
    ```
- Change `DEBUG` setting in `settings.py` to either `True` or `False`. Turn off in production.
- Install packages from `requirements.txt` from another project / repo
    ```bash
    # Ensure conda env exists e.g my_conda_env
    # If not create it: conda create -y --name my_conda_env
    conda install --force-reinstall -y --name my_conda_env -c conda-forge --file requirements.txt
    ```

## Imports
- Locate the python path in your virtual environment (with the virtual env activated)
	```bash
	which python
	```
- `ctrl + shift + p` in VS code.
- Launch workspace settings. In the `settings.json` file add the following line
	```JSON
	{
		"python.pythonPath": "/result/of/first/command",
	}
	```

## Env_variables
- Create `.env` file in the project's root folder
- Format:
	```text
	export VAR_NAME='Value'
	```
- Use `environ` package. `conda install -c conda-forge django-environ`
- Ensure `.env` is included in the `.gitignore` file. If not and the file has been pushed to the repo, remove it from git history using:
	```bash
	git rm -rf --cached --ignore-unmatch .env
	```
- [List of useful `.gitignore` files](https://github.com/github/gitignore)

## Templates:
- Create templates dirctory in the project root, then subdirectories for each app inside it.
- Edit `DIR` key of the `templates` dictionary inside of the project's `settings.py`
    ```python
    # Find way to do this using pathlib library instead
    import os
    ```
    ```python
    #...
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
    #...
    # Note the directory paths
    ```
- Create an `index.html` file inside the templates folder, add template variables appropriately.
- Template variable names must match keys defined in each app's `views.py`
- The `render` function's 2nd argument in `views.py` is the template file you want the content to appear in.
- Reference the `block` in `base.html` or whichever template of interest

## Static-files
- Create `static` directory in project root
- Add path to `settings.py`
- Add `STATIC_URL` variable
    ```python
    # Define STATIC_DIR variable
    STATIC_DIR = #....
    # ...
    STATICFILES_DIRS = [
        STATIC_DIR,
    ]
    ```
- Add template variables
    ```html
    <!-- Inside ./templates/base.html -->
    {% load static %}
    ```
    ```html
    <!-- Inside html -->
    <img src="{% static 'path/to/img.jpg' %}">
    ```
- Ideally each app will have its own static assests folder inside the `static` directory

## Models:
- To connect to different back-ends, edit / add the `DATABASES.default.ENGINE` and `DATABASES.default.NAME` variables.
- Models' syantax, inside each app's `model.py` file:
    ```python
    class <ClassName>(models.Model):
        #.... attributes
    ```
- Upon editing the models, register and make migrations:
    ```shell
    python manage.py migrate
    ```

    ```shell
    python manage.py makemigrations <app_name>
    ```
- To interact with the db via the shell
    ```shell
    python manage.py shell
    ```
- To make models available to the admin interface, register the models in the app's `admin.py` file:
    ```python
    # Import all the models from app
    #...
    admin.site.register(<ModelName>)
    # Register each model individually
    ```
- Populate the models and import them into the `views.py` so they can be displayed in the templates

## Population-script
- Populate db with fake data based on the defined models
    ```shell
    # within the activated environment
    pip install Faker
    ```
- Create a `populate_<app_name>.py` file, pref. in the project root.
    ```python
    import os
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', '<app_name>')

    import django
    django.setup()

    import random
    from faker import Faker
    from <app_name>.models import # all the models

    fake_gen = Faker()
    #... Data to be randomized based on models
    # Use fak_gen methods available to generate data for each field in a model
    # NOTE: when using foreign keys, use the entire object generated by the get_or_create() method for the corresponding field
    ```
- Run the population script:
    ```shell
    python populate_<app_name>.py
    ```
## Databases
- Avoid `sqlite` due to threading contraints. Multiple migration may result in a locked database.
- MySQL setup, assuming its already installed. If not use [this link to install](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04):
    ```shell
    # Install MySQL Database Connector
    sudo apt install python3-dev libmysqlclient-dev default-libmysqlclient-dev
    ```
    ```shell
    # make sure environment is active
    pip install mysqlclient
    ```
    ```shell
    sudo mysql -u root
    ```
    ```sql
    -- Don't forget the semi-colon at the end
    CREATE DATABASE <db_name>;
    ```
    ```shell
    python manage.py createsuperuser
    ```
- Inside `setting.py`
    ```python
    # ...
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'OPTIONS': {
                'read_default_file': '/etc/mysql/my.cnf',
            },
        }
    }
    #...
    ```
    On the command line:
    ```shell
    sudo gedit /etc/mysql/my.cnf
    ```
    Inside the file:
    ```sh
    [client]
    database = <db_name>
    user = <db_user>
    password = <db_password>
    default-character-set = utf8
    ```
    ```shell
    sudo systemctl daemon-reload; sudo systemctl restart mysql
    ```
- Test the connection to the application
    ```shell
    python manage.py migrate
    ```

## Forms
- Use `ModelForm` preferably to connect models to forms
- Import models into `forms.py` and add them to the `Meta` subclass using `fields = '__all__'` or whatever is appropriate.
    ```python
    from .models import MyModel1, MyModel2 #,...
    #....
    class MyForm(forms.ModelForm):
        # validators & custom fields go here
        class Meta:
            model = MyModel1
            fields = '__all__'
            exclude = # array of fields names to exclude
    ```
- Ensure models are registered in `admin.py`
- Create template file to render the form in the app's template folder
- In `views.py` import necessary models, then 
    ```python
    def form_handler(request):
        # Instantiate an instance of the form class
        form = MyForm()

        # Check for 'POST' method on the form
        if request.method == 'POST':
            # re-instantiate form class with form values
            form = MyForm(request.POST)
            # Ensure for is valid
            if form.is_valid():
                # Proceed to process form values
                pass
            else:
                # Handle validation errors
                pass
        
        return render(request, 'path/2/template.html', {'form': form})
    ```
- Add the above view to the app's `urls.py` file.

## Relative-URLs
- Syntax
    ```html
    <!-- url_name as defined in the app_name's urls.py file -->
    <a href="{% url 'url_name' %}">
        <!-- Content -->
    </a>
    <!-- OR app e.g admin-->
    <a href="{% url 'app_name:url_name' %}">
        <!-- Content -->
    </a>
    ```
## Template-filters
- Transform values of variables & tag args. Syntax:
    ```python
    {{ variable | filter_name }}
    # or
    {{ variable | filter_name: "parameter" }}
    ```
- Custom template filters inside `app/templatetags/mytemplates.py` file. (`templatetags` folder has an `__init__.py` file):
    ```python
    from django import template
    # Register templates
    register = template.Library()

    def my_func(value, arg):
        """
        Doc string explaining function in relation to "arg" & "value"
        """
        # Do something to "arg"

    register.filter('my_filter_name', my_func)
    ```
- Better yet with decorators: 👇️👇️👇️👇️
    ```python
    from django import template
    # Register templates
    register = template.Library()

    @register(name='filter_name')
    def my_func(value, arg):
        """
        Doc string explaining function in relation to "arg" & "value"
        """
        # Do something to "arg"

## Auth0-AuthZ
- Password hashing using bcrypt or Argon 2
    ```shell
    conda install -c conda-forge bcrypt
    # Or
    conda install -c conda-forge argon2-cffi
    ```
- Add them to `settings.py`
    ```python
    PASSWORD_HASHERS = [
        'django.contrib.auth.hashers.Argon2PasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.BCryptPasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    ]
    ```
- Import the built in `User` object
- Features: Username, Email, Password, First Name, Surname. 
- Attributes: is_active, is_staff, is_superuser etc. Can have custom ones.
    ```python
    from django.contrib.auth.models import User
    #...
    class UserInfo(model.Models):
        # create relationship, never inherit
        user = models.OneToOneField(User)
        # ....
    ```
- Register the model in `admin.py`
    ```python
    from .models import MyUserModel
    #...
    admin.site.register(MyUserModel)
    ```
- Inside `forms.py`
    ```python
    from django import forms
    from django.contrib.auth.model import User
    from .models import MyUserModel

    class MyUserForm(forms.ModelForm):
        # Add password field
        password = forms.CharField(widget=forms.PasswordInput())

        class Meta():
            model = User
            # Add other fields from django's User model. See docs
            # username, first_name, last_name, email
            fields = (..., 'password')
            #...

    class UserModelForm(forms.ModelForm):
        # add extrafields here, 3rd party package fields e.g cloudinary
        new_field_1 = forms.#....
        new_field_2 = forms.#...

        class Meta():
            model = MyUserModel
            fields = ('new_field_1', 'new_field_2',)
    ```

## [Back To Top](#Content)
