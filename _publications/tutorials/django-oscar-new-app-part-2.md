---
layout: publication
title: Creating an Oscar App with Dashboard (Part 2)
tags: django oscar app dashboard
category: Creating an Oscar App with Dashboard
date: 2021-03-17
output: true
description: Learning Django-Oscar can be cumbersome. In this tutorial, you will learn how to create a brand new Oscar app with a dashboard.
---


# Introduction
In this tutorial, you are going to learn how to create a dashboard for *Oscar* e-commerce framework. This tutorial starts where the [**first part**](/publications/tutorials/django-oscar-new-app-part-1/) ended.
## Creating a Django-Oscar dashboard app to manage boutiques  
Let's create a new app called dashboard inside the boutique app directory:

```
mkdir boutique/dashboard
```

Then initialize a new Django app using the following command:  

    python manage.py startapp dashboard boutique/dashboard
   
> You can delete `admin.py`, `models.py` and `tests.py`, because these are not required for the Oscar's dashboard app.
    
Once again after the dashboard app is created, it is necessary to register the app in `INSTALLED_APPS` in `settings.py` as shown below:

{% highlight python %}

INSTALLED_APPS = [
    ...
    'boutique.dashboard.apps.DashboardConfig',
]
{% endhighlight %}

> If you run server at this moment it will not work as you need to first complete the app configurations.

In the first part, we had a commented-out line in our `myoscarproject/urls.py`. Now that the dashboard app is created we need to uncomment it as show below:

{% highlight python %}

from django.apps import apps
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    ...
    path('dashboard/boutique/', apps.get_app_config('boutique_dashboard').urls),
    ...
]
{% endhighlight %}

However, at this point label, `boutique_dashboard` is not associated with any configuration. Therefore, let's move on and create the Boutique Dashboard Oscar app config. 

## App configs for the Boutique's dashboard

Configuration for boutique dashboard app is similar to configs from the first part of this tutorial. With few additions as shown below:

{% highlight python %}
from django.urls import path
from oscar.core.application import OscarDashboardConfig
from oscar.core.loading import get_class

class DashboardConfig(OscarDashboardConfig):
    name = 'boutique.dashboard'
    label = 'boutique_dashboard'
    namespace = 'boutique-dashboard'

    default_permissions = ['is_staff']

    def ready(self):
        self.boutique_list_view = get_class(
            'boutique.dashboard.views', 'DashboardBoutiqueListView')
        self.boutique_create_view = get_class(
            'boutique.dashboard.views', 'DashboardBoutiqueCreateView')
        self.boutique_update_view = get_class(
            'boutique.dashboard.views', 'DashboardBoutiqueUpdateView')
        self.boutique_delete_view = get_class(
            'boutique.dashboard.views', 'DashboardBoutiqueDeleteView')

    def get_urls(self):
        urls = [
            path('', self.boutique_list_view.as_view(), name='boutique-list'),
            path('create/', self.boutique_create_view.as_view(),
                 name='boutique-create'),
            path('update/<int:pk>/', self.boutique_update_view.as_view(),
                 name='boutique-update'),
            path('delete/<int:pk>/', self.boutique_delete_view.as_view(),
                 name='boutique-delete'),
        ]
        return self.post_process_urls(urls)
{% endhighlight %}

One important point in this configuration is to change the `label` parameter. The Django Oscar's default dashboard app conflicts with `DashboardConfig` of the Boutique dashboard app. Django's documentation state that: 

> "AppConfig.label defaults to the last component of `name`."

Therefore, it is necessary choose a different label like `boutique_dashboard` in order to "tell" Django that this dashboard app is different from Oscar's built-in dashboard app. 

Another difference between dashboard app config from primary boutique app config is the `default_permissions` parameter. This parameter sets the Oscar's dashboard permissions for this dashboard app. Since the Oscar has multiple user permission levels like one that has *Fulfilment Parters*, setting this parameter to `is_staff` disables access to this dashboard for any user except c users like super-users. 

## Forms for the Boutique's dashboard app

First, it is necessary to create forms for your custom dashboard app. Create a `forms.py` file in `boutique/dashboard` directory and add the following code:

{% highlight python %}
from django import forms
from django.db.models import Q
from django.utils.translation import gettext_lazy as _
from oscar.core.loading import get_model

Boutique = get_model('boutique', 'Boutique')

class DashboardBoutiqueSearchForm(forms.Form):
    name = forms.CharField(label=_('Boutique name'), required=False)
    city = forms.CharField(label=_('City'), required=False)

    def is_empty(self):
        d = getattr(self, 'cleaned_data', {})
        def empty(key): return not d.get(key, None)
        return empty('name') and empty('city')

    def apply_city_filter(self, qs, value):
        words = value.replace(',', ' ').split()
        q = [Q(city__icontains=word) for word in words]
        return qs.filter(*q)

    def apply_name_filter(self, qs, value):
        return qs.filter(name__icontains=value)

    def apply_filters(self, qs):
        for key, value in self.cleaned_data.items():
            if value:
                qs = getattr(self, 'apply_%s_filter' % key)(qs, value)
        return qs


class DashboardBoutiqueCreateUpdateForm(forms.ModelForm):
    class Meta:
        model = Boutique
        fields = ('name', 'manager', 'city')

{% endhighlight %}

In the code above `DashboardBoutiqueSearchForm` is a form to filter Boutique instances in the dashboard. We design our form so that it can filter by model's `city` and `name` fields. The form `DashboardBoutiqueCreateUpdateForm` is the create and update form required to create or edit a boutique instance. This form inherits Django's default `forms.ModelForm` so it is relatively simple to make it work.

## Views for the Boutique's dashboard app

There are four different views required to deploy a custom Oscar dashboard. These are:

- View to list boutique instances `DashboardBoutiqueListView`
- View to create a new boutique instance `DashboardBoutiqueCreateView`
- View to update/edit a boutique instance `DashboardBoutiqueUpdateView`
- View to delete a boutique instance `DashboardBoutiqueDeleteView`

Prior to moving on to the views add the following code to the head of a `views.py` file of the boutique's dashboard app:

{% highlight python %}
from django.contrib import messages
from django.template.loader import render_to_string
from django.urls import reverse_lazy
from django.utils.translation import gettext
from django.utils.translation import gettext_lazy as _
from django.views import generic
from oscar.core.loading import get_class, get_model

Boutique = get_model('boutique', 'Boutique')
BoutiqueCreateUpdateForm = get_class(
    'boutique.dashboard.forms', 'DashboardBoutiqueCreateUpdateForm')
DashboardBoutiqueSearchForm = get_class(
    'boutique.dashboard.forms', 'DashboardBoutiqueSearchForm')
{% endhighlight %}

### **Listing Boutique instances in the dashboard**

Listing boutique instances in a custom dashboard app is no different than any other Django app. The list view inherits Django's `generic.ListView` as shown in the following code:

{% highlight python %}
class DashboardBoutiqueListView(generic.ListView):
    model = Boutique
    template_name = "dashboard/boutique/boutique_list.html"
    context_object_name = "boutique_list"
    paginate_by = 20
    filterform_class = DashboardBoutiqueSearchForm

    def get_title(self):
        data = getattr(self.filterform, 'cleaned_data', {})

        name = data.get('name', None)
        city = data.get('city', None)

        if name and not city:
            return gettext('Boutiques matching "%s"') % (name)
        elif name and city:
            return gettext('Boutiques matching "%s" near "%s"') % (name, city)
        elif city:
            return gettext('Boutiques near "%s"') % (city)
        else:
            return gettext('Boutiques')

    def get_context_data(self, **kwargs):
        data = super().get_context_data(**kwargs)
        data['filterform'] = self.filterform
        data['queryset_description'] = self.get_title()
        return data

    def get_queryset(self):
        qs = self.model.objects.all()
        self.filterform = self.filterform_class(self.request.GET)
        if self.filterform.is_valid():
            qs = self.filterform.apply_filters(qs)
        return qs
{% endhighlight %}

The only non-trivial part of the code above is the additional parameter `filterform_class`, which is essentially a parameter that is recognized and processed by Oscar's templates. 

### **Creating Boutique instances in the dashboard** 

Similarly, the view responsible for creating the boutique instances inherits `generic.CreateView` and is shown in the following code:

{% highlight python %}
class DashboardBoutiqueCreateView(generic.CreateView):
    model = Boutique
    template_name = 'dashboard/boutique/boutique_update.html'
    form_class = BoutiqueCreateUpdateForm
    success_url = reverse_lazy('boutique-dashboard:boutique-list')

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx['title'] = _('Create new boutique')
        return ctx

    def forms_invalid(self, form, inlines):
        messages.error(
            self.request,
            "Your submitted data was not valid - please correct the below errors")
        return super().forms_invalid(form, inlines)

    def forms_valid(self, form, inlines):
        response = super().forms_valid(form, inlines)

        msg = render_to_string('dashboard/boutique/messages/boutique_saved.html',
                               {'boutique': self.object})
        messages.success(self.request, msg, extra_tags='safe')
        return response
{% endhighlight %}

In the code above, the parameter `success_url` is assigned to `reverse_lazy` and not `reverse` because the URL will be evaluated lazily(or when required). Moreover, Oscar uses Django's built-in messages framework to pass success and fail messages to the templates. The messages are handled in corresponding methods `forms_invalid` and `forms_valid`. 

### **Updating Boutique instances in the dashboard** 

View for updating Boutique instance is very similar to create a view and uses the same template but inherits `generic.UpdateView` instead.

{% highlight python %}
class DashboardBoutiqueUpdateView(generic.UpdateView):
    model = Boutique
    template_name = "dashboard/boutique/boutique_update.html"
    form_class = BoutiqueCreateUpdateForm
    success_url = reverse_lazy('boutique-dashboard:boutique-list')

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx['title'] = self.object.name
        return ctx

    def forms_invalid(self, form, inlines):
        messages.error(
            self.request,
            "Your submitted data was not valid - please correct the below errors")
        return super().forms_invalid(form, inlines)

    def forms_valid(self, form, inlines):
        msg = render_to_string('dashboard/boutique/messages/boutique_saved.html',
                               {'boutique': self.object})
        messages.success(self.request, msg, extrforms_valida_tags='safe')
        return super().forms_valid(form, inlines)
{% endhighlight %}

### **Deleting Boutique instances from the dashboard**

Delete view is rather simple compared to others and it inherits Django's `generic.DeleteView` as shown below:

{% highlight python %}
class DashboardBoutiqueDeleteView(generic.DeleteView):
    model = Boutique
    template_name = "dashboard/boutique/boutique_delete.html"
    success_url = reverse_lazy('boutique-dashboard:boutique-list')
{% endhighlight %}

Finally, now that views are completed we can move on to templates.

## Templates for the Boutique's dashboard app

For templates let's first create a directory `/src/templates/dashboard`. In this directory, we must implement three-view templates and one message template:

- Template for list view: `/dashboard/boutique/boutique_list.html`
- Template for update view: `/dashboard/boutique/boutique_update.html`
- Template for delete view: `/dashboard/boutique/boutique_delete.html`
- Message template: `/dashboard/boutique/messages/boutique_saved.html`

Templates are implemented the same way as was described in the first part of this tutorial except that these templates must extend different base layout,{% raw %}`{% extends 'oscar/dashboard/layout.html' %}`{% endraw %}.  Since templates are long you can find them in the [Git repository](https://github.com/mmtechslv/tutorial-django-oscar-newapp) of this tutorial. After templates are ready the following screen will be available when you go to `http://127.0.0.1:8000/dashboard/boutique/` URL:

<p align="center">
  <img width="620" height="233" src="/public/tutorials/django-oscar-new-app/OscarBoutiquesDashboardList.png">
</p>

## Adding "Boutiques" navigation item to Django-Oscar's dashboard

Finally, after *Boutqiues* are ready we need to add a navigation item to Oscar's dashboard navigation. Luckily, Django-Oscar provides a very easy way to do this. You need to add the following code to the settings but make sure that it comes after importing Oscar's defaults:

{% highlight python %}
from django.utils.translation import gettext_lazy as _

... # Django's Other Settings

from oscar.defaults import *

OSCAR_DASHBOARD_NAVIGATION.append({
    'label': _('Boutiques'),
    'icon': 'fas fa-store',
    'url_name': 'boutique-dashboard:boutique-list',
})
{% endhighlight %}

Once the navigation item is added you will get the following screen when entering your Oscar Dashboard:

<p align="center">
  <img width="511" height="113" src="/public/tutorials/django-oscar-new-app/OscarBoutiquesDashboardNavBar.png">
</p>

# Conclusion

At the end of this tutorial, you should be able to create a brand new Django Oscar app with a working dashboard and everything. I hope this tutorial was helpful for the reader and made one's life easier while learning such an amazing e-commence framework like Django-Oscar. 

> Source code of this tutorial can be found in my Git repository **[here](https://github.com/mmtechslv/tutorial-django-oscar-newapp)**







