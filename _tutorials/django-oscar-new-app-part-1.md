---
layout: collection
collection: tutorials
title: Creating an Oscar App with Dashboard (Part 1)
category: Creating an Oscar App with Dashboard
date: 2021-03-16
excerpt: Learning Django-Oscar can be cumbersome. In this tutorial, you will learn how to create a brand new Oscar app with a dashboard.
tags:
    - django
    - oscar
    - app
    - dashboard
header:
    teaser: /public/assets/tutorials/oscar-dashboard-new-app.jpg
redirect_from:
  - /publications/tutorials/django-oscar-new-app-part-1/
---


# Introduction
In this tutorial you are going to learn how to create a new Django app and integrate it into *Oscar* e-commerce framework. Particularly, we will create a new sample Django app called `boutique` and integrate it to the Oscar's default front and dashboard.  
## Getting ready (django-oscar)
First it is necessary to create a virtual environment to work in. I use *pipenv* as virtual environment for its simplicity and ease of use. Create a directory called `/myoscarapp`, move inside and run following command:

{% highlight bash %}

    $ pipenv shell
{% endhighlight %}

Then install the *django-oscar* using pip:

{% highlight bash %}

    $ pip install django-oscar[sorl-thumbnail]
{% endhighlight %}

Now create a brand new Dango project using following command and rename the created directory to `src` for convenience:

{% highlight bash %}

    $ django-admin startproject myoscarproject
    $ mv myoscarproject src
{% endhighlight %}

Next configure Django `settings.py` and `urls.py` as described in Oscar's [corresponding documentation](https://django-oscar.readthedocs.io/en/stable/internals/getting_started.html#django-settings).
Run `makemigrations` and `migrate`:

{% highlight bash %}

    python manage.py makemigrations
    python manage.py migrate
{% endhighlight %}

Test the website:

{% highlight bash %}

    python manage.py runserver
{% endhighlight %}

Following screen should be now available:
<p align="center">
  <img width="682" height="312" src="/public/tutorials/django-oscar-new-app/OscarInitial.png">
</p>


## Creating "boutique" app for Django-Oscar
New app is created as usual using following command:

    python manage.py startapp boutique

Once again as usual after the app is created, it is necessary to register the app in `INSTALLED_APPS` in `settings.py` as shown below:

{% highlight python %}

    INSTALLED_APPS = [
        ...
        'boutique.apps.BoutiqueConfig',
    ]
{% endhighlight %}

Similarly, your `urls.py` should look like this:

{% highlight python %}

    from django.apps import apps
    from django.urls import include, path
    from django.contrib import admin

    urlpatterns = [
        path('i18n/', include('django.conf.urls.i18n')),
        path('admin/', admin.site.urls),
        #path('dashboard/boutique/', apps.get_app_config('boutique_dashboard').urls),
        path('boutique/', apps.get_app_config('boutique').urls),
        path('', include(apps.get_app_config('oscar').urls[0])),
    ]
{% endhighlight %}

In the code above, line with `boutique_dashboard` URL configuration is temporarily commented out and will be turned on when Oscar's dashboard app is  forked.  
### Models for "boutique" app
Create following model that will represent a single **boutique** with three fields.

{% highlight python %}

    from django.db import models

    class Boutique(models.Model):
        name = models.CharField(max_length=255, blank=True, null=True)
        manager = models.CharField(max_length=150, blank=True, null=True)
        city = models.CharField(max_length=150, blank=True, null=True)

        class Meta:
            app_label = 'boutique'
{% endhighlight %}

### App configs for "boutique" app
While usual Django app's config class in `apps.py` inherits Django's default `django.apps.AppConfig`class, Oscar app's must inherit `oscar.core.application.OscarConfig` instead. Your `apps.py` should look like this:

{% highlight python %}

    from oscar.core.application import OscarConfig
    from django.urls import path, re_path
    from oscar.core.loading import get_class

    class BoutiqueConfig(OscarConfig):
        name = 'boutique'
        namespace = 'boutique'

        def ready(self):
            super().ready()
            self.boutique_list_view = get_class(
                'boutique.views', 'BoutiqueListView')
            self.boutique_detail_view = get_class(
                'boutique.views', 'BoutiqueDetailView')

        def get_urls(self):
            urls = super().get_urls()
            urls += [
                path('', self.boutique_list_view.as_view(), name='index'),
                re_path(r'^view/(?P<pk>\d+)/$',
                        self.boutique_detail_view.as_view(), name='details'),
            ]
            return self.post_process_urls(urls)
{% endhighlight %}

It is optional to use `get_class` and `get_model` when developing your own app but required when overriding Oscar apps. However, I prefer using Oscar's approach in all cases as I previously encountered various errors when importing modules using `import` statement.

### Admin for "boutique" app
This step is optional and Oscar's dashboard is sufficient to add, modify and remove `Boutique` elements to the database. However, for early testing let's register our model in Django's admin. Add following code to the `admin.py` in the app's directory.

{% highlight python %}

    from django.contrib import admin
    from oscar.core.loading import get_model

    Boutique = get_model('boutique', 'Boutique')

    class BoutiqueAdmin(admin.ModelAdmin):
        pass

    admin.site.register(Boutique, BoutiqueAdmin)
{% endhighlight %}

Now that the model is registered in Django's admin, go on and add few items for testing.

> To access Django's admin you will need to create a super user using command `python manage.py createsuperuser`

### Views for "boutique" app
There is nothing special in implementation of views that will deliver context to the front pages. Following is a working `views.py` based on Django's generic class-based views.

{% highlight python %}

    from django.views import generic
    from oscar.core.loading import get_model

    Boutique = get_model('boutique', 'Boutique')

    class BoutiqueListView(generic.ListView):
        model = Boutique
        template_name = 'boutique/boutique_list.html'
        context_object_name = 'boutique_list'

    class BoutiqueDetailView(generic.DetailView):
        model = Boutique
        template_name = 'boutique/boutique_details.html'
        context_object_name = 'boutique'
{% endhighlight %}

### Front-end templates for "boutique" views
First and foremost, let's override Oscar's navigation template by adding URL to our `BoutiqueListView`. First create a directory called `oscar` in in `/src/templates` directory. Any template file with same relative path Oscar's templates from source code will be overridden by Oscar and become a higher priority template. Because Oscar is developed in a very smart and customizable way, it is very easy to add an element to original Oscar template navigation. The original template HTML file from Oscar's source code can be found in `/templates/oscar/partials/nav_primary.html`. Accordingly, we need to create file `oscar/partials/nav_primary.html` that will contain following code:

{% highlight html+django %}{% raw %}

    {% extends "oscar/partials/nav_primary.html" %}
    {% load i18n %}

    {% block nav_items %}
    {{ block.super }}
    <li class="nav-item dropdown">
      <a class="nav-link" href="#" role="button">
        {% trans "Boutiques" %}
      </a>
    </li>
    {% endblock %}
{% endraw %}{% endhighlight %}

In the code above, we first extend the original Oscar's template. Then we override the block `nav_items`  by adding new element to the Oscar's default front-end navigation. After restarting the server, following front should show up:
<p align="center">
  <img width="416" height="195" src="/public/tutorials/django-oscar-new-app/OscarBoutiquesNavBar.png">
</p>

### Template for list of boutiques
Previously we created a view `BoutiqueListView`, which is responsible for delivering the context with a list of `Boutique` instances to the template `boutique/boutique_list.html`. Therefore, we first create an HTML file `/src/templates/boutique/boutique_list.html`. Notice that this template file is not placed under `/src/templates/oscar` directory. This is because we do not override Oscar's template and merely creating a new custom template. However, in our case, it does extend the default Oscar layout template as shown:

{% highlight html+django %}{% raw %}

    {% extends "oscar/layout.html" %}

    {% load i18n %}
    {% load product_tags %}

    {% block title %}
    {% trans "Boutiques" %} | {{ block.super }}
    {% endblock %}

    {% block breadcrumbs %}
        <nav aria-label="breadcrumb">
            <ol class="breadcrumb">
                <li class="breadcrumb-item">
                    <a href="{{ homepage_url }}">{% trans "Home" %}</a>
                </li>
                <li class="breadcrumb-item active" aria-current="page">{% trans "Boutiques" %}</li>
            </ol>
        </nav>
    {% endblock %}

    {% block headertext %}
        {% trans "Boutique" %}
    {% endblock %}

    {% block content %}
        {% if not boutique_list %}
            <p>{% trans "There are no boutique at the moment." %}</p>
        {% else %}
            {% for boutique in boutique_list %}
            <p>
              <h2><a href="{% url 'boutique:details' boutique.pk %}">{{ boutique.name }}</a></h2>
              The boutique is in: {{ boutique.city }}
            </p> <hr/>
            {% endfor %}
        {% endif %}
    {% endblock content %}
{% endraw %}{% endhighlight %}

The result should look like this:
<p align="center">
  <img width="480" height="252" src="/public/tutorials/django-oscar-new-app/OscarBoutiquesList.png">
</p>
### Template for boutique details
Now that we have a page with a list our boutique elements let's add page where user can view details of any given boutique. Similarly to the listing template, let's create a new HTML file `/src/templates/boutique/boutique_details.html` with following code:

{% highlight html+django %}{% raw %}

    {% extends "oscar/layout.html" %}

    {% load i18n %}
    {% load product_tags %}

    {% block title %}
    {% trans "Boutiques" %} | {{ block.super }}
    {% endblock %}

    {% block breadcrumbs %}
        <nav aria-label="breadcrumb">
            <ol class="breadcrumb">
                <li class="breadcrumb-item">
                    <a href="{{ homepage_url }}">{% trans "Home" %}</a>
                </li>
                <li class="breadcrumb-item" aria-current="page">
                  <a href="{% url 'boutique:index' %}">{% trans "Boutiques" %}</a>
                </li>
                <li class="breadcrumb-item active" aria-current="page">{{ boutique.name }}</li>
            </ol>
        </nav>
    {% endblock %}

    {% block headertext %}
        {% trans "Boutique" %}
    {% endblock %}

    {% block content %}
        <p>
          <h2>{{ boutique.name }}</h2> <br>
          The boutique is in: {{ boutique.city }} <br>
          The boutique's manager is Mr/Mrs: <strong>{{ boutique.manager }} </strong>
        </p>
    {% endblock content %}
{% endraw %}{% endhighlight %}

Result should look like this:
<p align="center">
  <img width="528" height="243" src="/public/tutorials/django-oscar-new-app/OscarBoutiquesDetails.png">
</p>

At this point the app's model, configs and its front-end templates are ready.  In the [**next**](/publications/tutorials/django-oscar-new-app-part-2/) tutorial we will develop Oscar dashboard for the `boutique` app.

> Source code of this tutorial can be found in my Git repository **[here](https://github.com/mmtechslv/tutorial-django-oscar-newapp)**
