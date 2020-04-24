# Using django-adminlte3 in a Project

`django-adminlte3` is a [Python](https://www.python.org/) module which "provides the functionality of the [AdminLTE3](https://adminlte.io/) theme to [Django](https://www.djangoproject.com/) developers in the form of standard base templates. Optional styling for Django's built-in admin interface is also provided."

From https://pypi.org/project/django-adminlte3/

In short, it lets you make your Django projects look very pretty with a consistent looking UI for minimal effort.  Kudos to the AdminLTE folks for the template, and to the Python dev who created this module.

For documentation of the module, see https://django-adminlte2.readthedocs.io

Lets get to it.
____
NOTE 1:  This is by no means a Django tutorial.  For that I suggest starting with the [official Django documentation](https://docs.djangoproject.com) which includes a nice tutorial.  There are also a ton of useful books, online course, YouTube videos, etc., which can cover the basics far better than I can.  The assumption here is that you have some familiarity with Django.

NOTE 2: This repo contains all the content explained below, minus the files/folders created by the `virtualenv` command.  So you could clone this repo, then follow step #1 starting with the `virtualenv .` line to get all the files needed to tinker.  But I explain the steps to build a Django project that uses this module from the ground up so you can do this yourself.  Note template files ending in `.example` are basically there so that, if you want to try them out, you simply rename the file by removing that extension, then reloading your browser to see how having that customized template affects things.

____
## Setup Project Environment

To build a Django application which utilizes the `django-adminlte3` pip module, here are the general steps:

1. Create a folder for your Django project (e.g., **Django-AdminLTE3**) and build virtual environment with pip modules:
```bash
mkdir Django-AdminLTE3
cd Django-AdminLTE3
# create venv environ, which adds a ./bin, ./include, and ./lib directories
virtualenv .
source bin/activate
pip list
# pip install -r requirements.txt
# Or just do the following:
pip install django
pip install django-adminlte3
```

2. Create Django project (e.g., **TestAdminLTE3**) and your app (e.g., **TestApp**)
```bash
django-admin startproject TestAdminLTE3
cd TestAdminLTE3
django-admin startapp TestApp
```

3. Run migrations
```
python3 manage.py migrate
```

4. Create superuser account (so you can access Django admin pages):
```
python3 manage.py createsuperuser
```

Fill in username and password as needed.  Email optional.

5. Run server to verify things are working
```
python3 manage.py runserver
```

6.  Load the default page (what typically is http://127.0.0.1:8000) to verify things are working.

7.  Load the admin page (what typically is http://127.0.0.1:8000/admin) to see what the default looks like before adding AdminLTE to the mix.  Login and make note of how things look. Then logout (link in upper right) and click the upper left to reload the Admin page itself.

8. Modify `Django-AdminLTE3/TestAdminLTE3/TestAdminLTE3/settings.py`

```python
...
INSTALLED_APPS = [
    # For apps which use customized AdminLTE templates stored in
    # <app>/templates/adminlte/lib/, be sure to place these apps
    # BEFORE adminlte3 or the customized templates won't be found
    'TestApp',

    # General use templates & template tags (should appear first before apps)
    'adminlte3',

    # Optional: Django admin theme (must be before django.contrib.admin)
    'adminlte3_theme',
...
```
From above in settings.py, Django will search in the order of the installed apps for a matching template.  This is why the order matters.  If you use customized AdminLTE template files in your app but it is located BELOW the `adminlte` app, your customized templates will never be reached.

9. Reload the admin page.  You should now see how AdminLTE affects the admin login page.  Login again and make note of how things look now. Pretty sweet!

If you've done the steps above, you have the pieces in place, and from the admin page you already have a taste for what AdminLTE can do.  Time to actually make use of this.

____
## Creating Your First Page

10. Now, if you want to simply get a taste for what this module can do for you, create a template page `index.html` in

`Django-AdminLTE3/TestAdminLTE3/TestApp/Templates/TestApp/`

(creating any subdirectories needed) which contains the following:

```html
{% extends 'adminlte/base.html' %}

<!-- Set title of app -->
{% block title %}TestApp{% endblock %}

<!-- Specify content of main body of page -->
{% block content %}
<h1>Test site using django-adminlte3 pip module.</h1>

<p>Let us see how this works.</p>

  For docs, check out <a href="https://pypi.org/project/django-adminlte-3/">docs</a>
{% endblock %}
```

So what's going on here?
* Just as with any Django templates, you're using Jinja2 formatting with the usual {% %} tags in your template file
* Line 1 extends the default AdminLTE `base.html` file included by the pip module, allowing you to override different blocks defined in that template.  (This is typically the minimum you'll want to do.)
* The two blocks, `title` and `content`, which are defined in base.html, are overridden, resulting in their contents appearing on the page.  These were chosen just to show how things fit together (think "hello world").  If you did not override these (i.e., if you deleted all but the first line), you'd see whatever was defined in the default AdminLTE template `base.html` file.

Of course, a template only works if there's a Django view accessing it.  So now create your first view.

11. Modify `Django-AdminLTE3/TestAdminLTE3/TestApp/views.py` as follows:

```python
from django.shortcuts import render
from django.views import generic

# Create your views here.

class IndexView(generic.TemplateView):
    template_name = 'TestApp/index.html'
```

Here the Django generic TemplateView class was viewed to keep it simple.  This view simply loads that corresponding template file.

Next, we need to make sure Django has a URL pattern configured to reach this app's view.  Here we use the best practice of defining URL patterns within an app, which provides portability should we want to deploy this app in another Django project.

12. Create file `Django-AdminLTE3/TestAdminLTE3/TestApp/urls.py` to define URL patterns to use in the TestApp app:

```python
from django.urls import path
from . import views

app_name = 'TestApp'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
]
```

Since there's just one view, `index.html`, we specify the URL pattern `''` so it's the home/index page for the TestApp.

13. Modify the main Django project file `Django-AdminLTE3/TestAdminLTE3/TestAdminLTE3/urls.py` to stitch the TestApp URL patterns into the overall Django project URL patterns:

```python
from django.contrib import admin
from django.urls import include,path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('TestApp.urls')),
]
```

Here, since the only app is TestApp, and we want it to load on the main page of the project, we specify `''` as the URL pattern.  This means visiting the main page (e.g., http://127.0.0.1:8000) maps to this URL pattern, which flows through to the TestApp `urls.py` file, which in turn maps to the IndexView, which then loads the TestApp `index.html` template file.

14.  Load the main page again (what typically is http://127.0.0.1:8000).  Voila!
15. Customize AdminLTE templates as needed.
16. Profit!

Ok ok, kidding.  So let's take this slow.

Now, that default page is nice to look at.  It contains various elements, from a header, footer, and side navigation bar, to a logo and a user panel, etc.  But none of them actually work.  So let's continue for just a little.

____
## Short Aside:  How Does This Work?

I like to understand how things work if I'm going to use them.  So in that spirit, for those similarly inclined, note that when you install the `django-adminlte3` Python module using pip, it gets installed like any other Python module into the `site-packages` directory.  For example, if you followed the steps above, take a look in

`Django-AdminLTE3/lib/python<version>/site-packages`

There you will find directories `adminlte3`, `adminlte3_theme`, `django`, and `django_adminlte3`, among others.  `adminlte3` contains the actual AdminLTE template files.  Note you don't modify these.  If you wish to customize things, you create your own versions, which you store in a path that matches the default ones (more on that in a minute).  And in these you place an `{% extend <file> %}` line at the top to pull that default file in and then modify it as needed.  But it's worth knowing where the default ones live in case you want to look at them.  Either that or download the official AdminLTE files and look through those for examples to work with.  To see examples of what the template offers, check out their [live preview page](https://adminlte.io/preview).

Now on with the show.

____
## Modifying the Sidebar

Next, let's say you wish to modify the sidebar, which is defined in

`./adminlte/lib/_main_sidebar.html`

To do so, recreate that path in your app, e.g.,

`Django-AdminLTE3/TestAdminLTE3/TestApp/templates/adminlte/lib/_main_sidebar.html`

and adjust this file as needed, such as
```html
{% extends 'adminlte/lib/_main_sidebar.html' %}
{% load static %}

<!-- Set upper left corner logo -->
{% block logo %}
    <a href="/" class="brand-link">
        <img src="{%static 'TestApp/logo.png' %}" alt="My Logo" class="brand-image img-circle elevation-3" style="opacity: .8">
        {% block logo_text %}<span class="brand-text font-weight-light">TestApp</span>{% endblock %}
    </a>
{% endblock logo %}

<!-- Remove user panel from left sidebar -->
{% block user_panel %}
{% endblock user_panel %}

<!-- Set header for side navbar -->
{% block nav_heading %}
WHERE TO GO
{% endblock nav_heading %}

<!-- Define navbar links -->
{% block nav_links %}
    <li>
        <a href="/">
            <i class="fa fa-home"></i>
            <span>Home</span>
        </a>
    </li>
    <li>
        <a href="/admin">
            <i class="fa fa-cog"></i>
            <span>Admin</span>
        </a>
    </li>
{% endblock nav_links %}
```

The file above
* Extends the default AdminLTE `_main_sidebar.html` file
* Sets a logo that appears in the upper left corner (NOTE:  For this to work, you must, of course, actually have a file named `logo.png` that is stored in `Django-AdminLTE3/TestAdminLTE3/TestApp/static/TestApp/`)
* Removes the default user_panel shown
* Sets the heading of the navbar
* Defines links for `Home` and `Admin`, setting the icons next to them to a home and cog, respectively

____
## Next Steps

Similar to the file `_main_sidebar.html`, there's also `_main_header.html` and `_main_footer.html`.  If you wish to modify those elements, simply create these files in the same directory as `_main_sidebar.html`, then edit them accordingly.

Note the [documentation for django-adminlte2](https://django-adminlte2.readthedocs.io) is a very good resource.  (Though the django-adminlte3 site claims to have documentation at a nearly identical site, just for version 3, that site doesn't exist.  But to date haven't had issues using the v2 docs to sort through things.)

____
## Where To Go From Here

AdminLTE offers a _lot_ more features than this, as can be seen on their site and in ther live preview.  But I'll leave you with one last tidbit.  Regarding the possible icons you can make use of, note above the class attributes `fa fa-home` and `fa fa-cog`.  To help you find the right icon to use and style, use this [search page on Font Awesome](https://fontawesome.com/icons?d=gallery).
