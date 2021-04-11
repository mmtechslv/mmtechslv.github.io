---
layout: collection
collection: tutorials
title: How To Create a Fancy Range-Slider Filter In Django
category: Django Tutorials
date: 2021-04-07
output: true
excerpt: In this tutorial, you will learn how to create a fancy range-slider using django-crispy-forms and django-filters packages.
tags:
    - django
    - range-slider
    - django-crispy-forms
    - django-filters
header:
    teaser: /public/assets/tutorials/django-crispy-range-slider.png
---

{% raw %}
# Introduction
In this tutorial, we are going to create a nice crispy range slider using django-crispy-forms for an integer filter provided by django-filters. Tutorial can be split into four sections. In the first or prerequisite section, we will install the required packages and initiate the Django project. In the next section, we will create the simple app with a model, view, and template. In the third section, we will create the simple range filter using *django-filters* package without a slider. In the fourth and last section, we will describe how to create a range-slider and integrate it into our Django app created in the third section.  

# Prerequisite

First things first, let's create a directory with the working environment:

```bash
$ mkdir myproject
$ cd myproject
$ pipenv shell
```

Then, install packages that are required in this tutorial using `pip`:

```bash
$ pip install Django
$ pip install django-crispy-forms
$ pip install django-filter
```

Next, create a new Django project called **myproject**:

```bash
$ django-admin startproject myproject
$ mv myproject src
```

Similarly, create a new Django app called **myapp**:

```bash
$ python manage.py startapp myapp
```

In the following sections, you are going to need to generate sample data for our model. Hence, let's create a new Django admin super-user using the following command:

```bash
$ python manage.py createsuperuser
```

To enable packages in Django project, add following lines to the `INSTALLED_APPS` in `/src/myproject/settings.py`:

```python
INSTALLED_APPS = [
    ...
    'django.forms', # Required in the last section.
    'django_filters', # Django-filter
    'crispy_forms',
    'myapp'
]
```

Then, add the following line to `TEMPLATES` in `/src/myproject/settings.py`:

```python
TEMPLATES = [
    {
        ...
        'DIRS': [BASE_DIR / 'templates'],
        ...
    },
]
```

Next, add path `/src/myproject/static` to `STATICFILES_DIRS` in `/src/myproject/settings.py` to enable the CSS and JS files, which will be required in upcoming sections:

```python
...
STATICFILES_DIRS = [ BASE_DIR / 'static']
...
```

Finally, add the following line of code to `/src/myproject/settings.py` to enable widget customization in our Django project.

```python
...
FORM_RENDERER = 'django.forms.renderers.TemplatesSetting'
...
```

# Getting Ready

In this section, we will create a model called `People`, a view for this model, and a template for that view.  

## The Model

Create a model called `People` using three fields **name**, **surname** and **age**. The target filter-field is `IntegerField` named **age**:

```python
class People(models.Model):
    name = models.CharField(null=True,blank=True,max_length=50)
    surname = models.CharField(null=True,blank=True,max_length=50)
    age = models.IntegerField()
```

Run `makemigrations` and then ``migrate`` to apply change to the default SQLite database:

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

Then, register the model in Django admin by adding the following code to file `/src/myapp/admin.py`:

```python
from django.contrib import admin
from .models import People

class PeopleAdmin(admin.ModelAdmin):
    pass

admin.site.register(People, PeopleAdmin)
```

> **_NOTE:_** Add some items to the database from the Django admin page at <http://127.0.0.1:8000/admin/>.

## The View

Now let's create a simple view that will print all instances of `People` model in `/src/myapp/views.py`.

```python
from django.shortcuts import render
from .models import People

def index(request):
    all_people = People.objects.all()
    return render(request, 'index.html', {'all_people':all_people})
```

## URLs

Create a URL path by adding following line to the file `/src/myproject/urls.py`:

```python
...
from myapp.views import index

urlpatterns = [
    ...
    path('', index),
]
```

## The Template

In order to create the simplest template to render all People instances, create a file `/src/templates/index.html` with the following content:

```
<table border='1' style="width:100%; text-align:center">
  <thead>
    <tr>
      <th> Name </th>
      <th> Surname </th>
      <th> Age </th>
    </tr>
  </thead>

  <tbody>
    {% for person in all_people %}
    <tr>
      <td> {{ person.name }} </td>
      <td> {{ person.surname }} </td>
      <td> {{ person.age }} </td>
    </tr>
    {% endfor %}
  </tbody>
</table>
```

## Recap
In this section, we created a simple view and template to print database records for `People` model. Executing `$ python manage.py runserver` should make available the following screen:

<p align="center">
  <img width="709" height="199" src="/public/tutorials/django-crispy-range-slider/all_people.png">
</p>

# Naive Range Filter

In order to ensure coherence let's first create a simple(or naive) `RangeFilter` provide by *django-filters* package.

## The Filter
Create in a new file `/src/myapp/filters.py` and insert following code:

```python
import django_filters
from .models import People

class PeopleFilter(django_filters.FilterSet):
  age = django_filters.AllValuesFilter()

  class Meta:
      model = People
      fields = ['age']
```

This will create a simple range-filter with minimum and maximum value requirements for `age` field of `People` model.

## The View
Now that filtering logic is ready, let's add the filtering feature to the main view in `/src/myapp/views.py`.

```python
from django.shortcuts import render
from .filters import PeopleFilter

def index(request):
    people_filter = RangeFilter(request.GET)
    return render(request, 'index.html', {'people_filter':people_filter})
```

In the above code, `RangeFilter` instantiation takes `request.GET` as a single parameter since our form is set to *GET* mode.

## The Template
With our filter ready, we can add filter controls in the front. Once again change the primary template file `/src/template/index.html` to look like this:

```
<form method="get">
    {{ people_filter.form.as_p }}
    <input type="submit" />
</form>

<table border='1' style="width:100%; text-align:center">
  <thead>
    <tr>
      <th> Name </th>
      <th> Surname </th>
      <th> Age </th>
    </tr>
  </thead>

  <tbody>
    {% for person in people_filter.qs %}
    <tr>
      <td> {{ person.name }} </td>
      <td> {{ person.surname }} </td>
      <td> {{ person.age }} </td>
    </tr>
    {% endfor %}
  </tbody>
</table>
```

Notice that we now have an additional filtering form provided by *django-filters*. In addition, observe that `for` statement loops over `people_filter.qs` instead of `all_people`. The `.qs` stands for query set, which is self-explanatory.

## Recap
In this section, we created the simplest or naive filter. The final result should look like this:

<p align="center">
  <img width="709" height="226" src="/public/tutorials/django-crispy-range-slider/simple_range_filter.png">
</p>

# Crispy Range-Slider

To create a working crispy range-slider we need the following:
 - Front-end Backbone: Actual range-slider with HTML, CSS and JavaScript.
 - Django Widget: Custom Django widget for the actual range-slider backbone.
 - Crispy Form: Crispy form with a layout template for the custom widget.
 - Range Filter: Custom filter from *django-filters* package that utilizes the range-slider widget with the Crispy form.  

Each point will be described in details so let's move step-by-step:

## The Front-End

The first and obvious step is to create an actual slider. Since range-slider is fancy filtering "thing" and not a real HTML form element, let's use a popular trick to make such a fancy "thing" act like an HTML form element. Particularly, we use [jQuery's range slider](https://jqueryui.com/slider/#range) feature to make our range-slider work. Here is a sample HTML blueprint for our slider:

```
<div id="my-numeric-slider"
     class="form-group numeric-slider"
     data-range_min="[Min. Possible Value]"
     data-range_max="[Max. Possible Value]"
     data-cur_min="[Current Min. Value]"
     data-cur_max="[Current Max. Value]">
<div class="numeric-slider-range ui-slider ui-slider-horizontal ui-slider-range"></div>
<span class="numeric-slider-range_text" id='my-numeric-slider_text'>[Lower Value] - [Upper Value]</span>
<input type='hidden' id='my-numeric-slider_min' name='slider-min'/>
<input type='hidden' id='my-numeric-slider_max' name='slider-max'/>
</div>
```

The above HTML markup is comprised of outer __Div__ element where the first two __data-__ attributes represent possible minimum and maximum values of the range filter, last two __data-__ attributes represent current lower and upper values of the range filter that were established when the page is loaded. Likewise, the first inner elements __Div__ with the class `numeric-slider-range` is the main element transformed to the range slider by jQuery when the page is loaded. The last two hidden __Input__ form-elements represent the primary means by which the data is passed from the client to the server-side when the form is submitted. Additionally, the above HTML markup requires a JS script to make the slider work and a CSS markup to render the elements properly. Both can be found in [GitHub repo](https://github.com/mmtechslv/tutorial-django-crispy-range-slider). Finally, apply the last template code bellow to the file `/src/templates/index.html`:

```
{% load static %}
{% load crispy_forms_tags %}

<head>
  <link rel="stylesheet" href="{% static 'custom_slider.css' %}"> # CSS of our range-slider.
  <link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">

  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
  <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
</head>

<body>
  {% crispy people_filter.form %}

  <table border='1' style="width:100%; text-align:center">
    <thead>
      <tr>
        <th> Name </th>
        <th> Surname </th>
        <th> Age </th>
      </tr>
    </thead>

    <tbody>
      {% for person in people_filter.qs %}
      <tr>
        <td> {{ person.name }} </td>
        <td> {{ person.surname }} </td>
        <td> {{ person.age }} </td>
      </tr>
      {% endfor %}
    </tbody>
  </table>

  <script src="{% static 'custom_slider.js' %}"></script> # JS of our range-slider.
</body>
```

Notice that in the HTML above we load the `crispy_forms_tags` and use `{% crispy people_filter.form %}` instead of `.as_p`. Doing so we let *django-crispy-forms* package handle our form rendering.

> **_NOTE:_** The *django-crispy-forms* package provides two ways to render crispy forms. Common way is to use `|crispy filter` to render the form but it expects to be wrapped in `<form>...</form>` HTML tag. In our tutorial, we use `{% crispy %}` tag because we will generate our form using the `FormHelper`.

## The Django Widget

Django uses widgets to render form-fields and present them as final HTML markup. Therefore, to let Django handle the rendering of our front-end backbone, we need a working widget. In Django, a widget consists of two parts:
- a widget-template that represents the final HTML
- a class that inherits Django's `Widget` class with `render()` method.

### Widget Class

The *django-filters* package provides a working `RangeFilter` with a predefined `RangeWidget` that we seamlessly/automagically used in the previous section. This widget uses two text-fields associated with two text-input HTML form-elements. Notice that it is similar to hidden input-elements required in our case. To make our widget work we will simply rewrite the default `RangeWidget` provided by *django-filter*  to be compatible with our range-slider. For simplicity let's call it the `CustomRangeWidget`:

```python
from django.forms.widgets import HiddenInput
from django_filters.widgets import RangeWidget

class CustomRangeWidget(RangeWidget):
    template_name = 'forms/widgets/range-slider.html'

    def __init__(self, attrs=None):
        widgets = (HiddenInput(), HiddenInput())
        super(RangeWidget, self).__init__(widgets, attrs)

    def get_context(self, name, value, attrs):
        ctx = super().get_context(name, value, attrs)
        cur_min, cur_max = value
        if cur_min is None:
            cur_min = ctx['widget']['attrs']['data-range_min']
        if cur_max is None:
            cur_max = ctx['widget']['attrs']['data-range_max']
        ctx['widget']['attrs'].update({'data-cur_min':cur_min,
                                        'data-cur_max':cur_max})
        base_id = ctx['widget']['attrs']['id']
        for swx, subwidget in enumerate(ctx['widget']['subwidgets']):
            subwidget['attrs']['id'] = base_id + "_" + self.suffixes[swx]
        ctx['widget']['value_text'] = "{} - {}".format(cur_min,cur_max)
        return ctx
```

### Widget Template

The widget also requires an associated template. Let’s create a file in `/src/templates/forms/widgets/range-slider.html` and insert the following content.

```
<div class="form-group numeric-slider" {% include "django/forms/widgets/attrs.html" %}>
  <div class="numeric-slider-range ui-slider ui-slider-horizontal ui-slider-range"></div>
  <span class="numeric-slider-range_text" id='{{ widget.attrs.id }}_text'>
    {{ widget.value_text }}
  </span>
  {% for widget in widget.subwidgets %}
    {% include widget.template_name %}
  {% endfor %}
</div>
```

In the above widget-template, we use `{% include "django/forms/widgets/attrs.html" %}` to let Django handle the widget attributes. It does so by parsing dictionary `ctx['widget']['attrs']` from previous part. Likewise, the `for` loop add widget's `HiddenInput` elements.

## The Crispy Form

At last, we have our actual widget ready and now we can create a crispy-form with a special template for our slider. This crispy layout template basically helps our widget to fit the Bootstrap markup logic. In other words, it makes it *crispy*.

### Crispy Template

Create a new file `/src/templates/forms/fields/range-slider.html`. Then add the following template code:

```
{% load crispy_forms_field %}

<div class="form-group{% if 'form-horizontal' in form_class %} row{% endif %}">
  <label for="{{ field.id_for_label }}" class="{% if 'form-horizontal' in form_class %}col-form-label {% endif %}{{ label_class }}{% if field.field.required %} requiredField{% endif %}">
    {{ field.label|safe }}
    {% if field.field.required %}
      <span class="asteriskField">*</span>
    {% endif %}
  </label>
  {% crispy_field field  %}
</div>
```
> **_NOTE:_** the above code is based on *django-crispy-forms*'s **bootstrap4** templates and was not tested in **bootstrap3** or other crispy template-engine.

### Crispy Form Helper
Once the crispy template is ready we need a form where the template will be utilized. Create a file `/src/myapp/forms.py` and add the following code:

```python
from crispy_forms.helper import FormHelper
from crispy_forms.bootstrap import StrictButton
from crispy_forms.layout import Field, Layout
from django import forms
from django_filters.fields import RangeField

class PeopleFilterFormHelper(forms.Form):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper(self)
        self.helper.form_method = 'get'
        layout_fields = []
        for field_name, field in self.fields.items():
            if isinstance(field, RangeField):
                layout_field = Field(field_name, template="forms/fields/range-slider.html")
            else:
                layout_field = Field(field_name)
            layout_fields.append(layout_field)
        layout_fields.append(StrictButton("Submit", name='submit', type='submit', css_class='btn btn-fill-out btn-block mt-1'))
        self.helper.layout = Layout(*layout_fields)
```

In the code above the class `PeopleFilterFormHelper` is nothing different than a simple Django form with a fancy name. However, instead of a common way of constructing Django form we use the Crispy approach with its `FormHelper`.

> **_NOTE:_** the `FormHelper` simply helps you to create a fancy form, which would most certainly be possible to create with the same Django means but with more effort. Our form is rather basic for the sake of clarity of our tutorial, so it is not obvious.


## Range Filter
At last, we have everything ready except the actual filtering logic.

### Custom Range Filter
Insert following final filter code to the file `/src/myapp/filters.py`.

```python
from django_filters import FilterSet
from django_filters.filters import RangeFilter
from .models import People
from .forms import PeopleFilterFormHelper
from .widgets import CustomRangeWidget

class AllRangeFilter(RangeFilter):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        values = [p.age for p in People.objects.all()]
        min_value = min(values)
        max_value = max(values)
        self.extra['widget'] = CustomRangeWidget(attrs={'data-range_min':min_value,'data-range_max':max_value})

class PeopleFilter(FilterSet):
  age = AllRangeFilter()

  class Meta:
      model = People
      fields = ['age']
      form = PeopleFilterFormHelper
```

In the code above, `PeopleFilter` represents the set of filters for model `People`. Notice the line `form = PeopleFilterFormHelper`, which overrides default form builder of *django-filters* to our custom `PeopleFilterFormHelper`. The actual filter is the `AllRangeFilter` class, which is a customized version of the original *django-filters* package's `RangeFilter`. We override its `__init__` method and initiate our custom  widget `CustomRangeWidget` with initial minimum and maximum values of all possible age values from `People` model's `age` field.  


> **_NOTE:_** I totally agree that list comprehension is far not the best way to get the min. and max. values but this is a tutorial and its for only for educational purpose.

## Recap
At last, we created our range filter with a fancy slider that should looks like this:

<p align="center">
  <img width="709" height="211" src="/public/tutorials/django-crispy-range-slider/fancy_range_slider.png">
</p>

# Summary
In this tutorial, you learned how to create a fancy jQuery range slider with the custom widget for custom range filter provided by `django-filters` package. Moreover, you learned how to use render the custom widget using `django-crispy-forms` package. The source code for this tutorial can be found on my [GitHub repo](https://github.com/mmtechslv/tutorial-django-crispy-range-slider). I hope this tutorial was helpful to the reader and eased his suffering while learning these amazing Django packages.
{% endraw %}
