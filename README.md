[![Build Status](https://travis-ci.org/Frojd/django-react-templatetags.svg?branch=master)](https://travis-ci.org/Frojd/django-react-templatetags)
[![PyPI version](https://badge.fury.io/py/django_react_templatetags.svg)](https://badge.fury.io/py/django_react_templatetags)

# Django-React-Templatetags

This django library allows you to add React components into your django templates.


## Index

- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Setup](#quick-setup)
- [Usage](#usage)
- [Settings](#settings)
- [Simple Example](#simple-example)
- [Working With Models](#working-with-models)
- [Server Side Rendering](#server-side-rendering)
- [FAQ](#faq)
- [Tests](#tests)
- [Contributing](#contributing)
- [License](#license)


## Requirements

- Python 2.7 / Python 3.4+ / PyPy
- Django 1.8+


## Installation

Install the library with pip:

```
$ pip install django_react_templatetags
```


## Quick Setup

Make sure `django_react_templatetags` is added to your `INSTALLED_APPS`.

```python
INSTALLED_APPS = (
    # ...
    'django_react_templatetags',
)
```

You also need to add the `react_context_processor` into the `context_middleware`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            'templates...',
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'debug': True,
            'context_processors': [
                ...
                'django_react_templatetags.context_processors.react_context_processor',
            ],
        },
    },
]
```

This should be enough to get started.


## Usage

1. Load the `{% load react %}`
2. Insert component anywhere in your template: `{% react_render component="Component" props=my_data %}`. This will create a dom placeholder.
3. Put `{% react_print %}` in the end of your template. (This will output the `ReactDOM.render()` javascript).


## Settings

- `REACT_COMPONENT_PREFIX`: Adds a prefix to your React.createElement include.
    - Example using (`REACT_COMPONENT_PREFIX="Cookie."`)
    - ...Becomes: `React.createElement(Cookie.MenuComponent, {})`
- `REACT_RENDER_HOST`: (SSR Only) Which endpoint SSR requests should be posted at. 
    - Example: `http://localhost:7000/render-component/`
- `REACT_RENDER_TIMEOUT`: (SSR Only) Timeout for SSR requests, in seconds.


## Simple example

This view...

```python
from django.shortcuts import render

def menu_view(request):
    return render(request, 'myapp/index.html', {
        'menu_data': {
            'example': 1,
        },
    })
```

... and this template:

```html
{% load react %}
<html>
    <head>...</head>

    <body>
        <nav>
            {% react_render component="Menu" props=menu_data %}
        </nav>
    </body>

    {% react_print %}
</html>
```

Will transform into this:

```html
<html>
    <head>...</head>

    <body>
        <nav>
            <div id="Menu_405190d92bbc4d00b9e3376522982728"></div>
        </nav>
    </body>

    <script>
        ReactDOM.hydrate(
            React.createElement(Menu, {"example": 1}),
            document.getElementById('Menu_405190d92bbc4d00b9e3376522982728')
        );
    </script>
</html>
```

## Working with models

In this example, by adding `RepresentationMixin` as a mixin to the model, the templatetag will know how to generate the component data. You only need to pass the model instance to the `react_render` templatetag.

This model...

```python
from django.db import models
from django_react_templatetags.mixins import RepresentationMixin

class Person(RepresentationMixin, models.Model):
    first_name = models.CharField(max_length=255)
    last_name = models.CharField(max_length=255)

    def to_react_representation(self, context={}):
        return {
            'first_name': self.first_name,
            'last_name': self.last_name,
        }
```

...and this view

```python
import myapp.models import Person

def person_view(request, pk):
    return render(request, 'myapp/index.html', {
        'menu_data': {
            'person': Person.objects.get(pk=pk),
        },
    })
```

...and this template:

```html
{% load react %}
<html>
    <head>...</head>

    <body>
        <nav>
            {% react_render component="Menu" props=menu_data %}
        </nav>
    </body>

    {% react_print %}
</html>
```

...will transform into this:

```html
...

<script>
    ReactDOM.hydrate(
        React.createElement(Menu, {"first_name": "Tom", "last_name": "Waits"}),
        document.getElementById('Menu_405190d92bbc4d00b9e3376522982728')
    );
</script>
```


## Server Side Rendering

This library supports SSR (Server Side Rendering) throught third-part library [Hastur](https://github.com/Frojd/Hastur).

It works by posting component name and props to endpoint, that returns the html rendition. Payload example:

```json
{
    "componentName": "MyComponent",
    "props": {
        "title": "my props title",
        "anyProp": "another prop"
    },
    "static": false
}
```

`REACT_RENDER_HOST` needs to be defined to enable communication with service.


## FAQ

### How do I override the markup generated by `react_print`?

Simple! Just override the template `react_print.html`

### This library only contains templatetags, where is the react js library?

This library only covers the template parts (that is: placeholder and js render).

### I dont like the autogenerated element id, can I supply my own?

Sure! Just add the param `identifier="yourid"` in `react_render`.

Example:
```
{% react_render component="Component" identifier="yourid" %}
```

...will print 
```html
<div id="yourid"></div>
```

### How do I pass individual props?

Add your props as arguments prefixed with `prop_*` to your `{% react_render ... %}`. 

Example: 
```html
{% react_render component="Component" prop_country="Sweden" prop_city="Stockholm" %}
```

...will give the component this payload:
```javascript
React.createElement(Component, {"country": "Sweden", "city": "Stockholm"}),
```

### How do I apply my own css class to the autogenerated element?
    
Add `class="yourclassname"` to your `{% react_render ... %}`. 
    
Example: 
```html
{% react_render component="Component" class="yourclassname" %}
```

...will print 
```html
<div id="Component_405190d92bbc4d00b9e3376522982728" class="yourclassname"></div>
```

### I want to pass the component name as a variable, is that possible?

Yes! Just remove the string declaration and reference a variable in your `{% react_render ... %}`, the same way you do with `props`.

Example:

This view

```python
render(request, 'myapp/index.html', {
    'component_name': 'MegaMenu',
})
```

...and this template

```html
{% react_render component=component_name %}
```

...will print:

```html
<div id="Component_405190d92bbc4d00b9e3376522982728" class="yourclassname"></div>
React.createElement(MegaMenu),
```


## Tests

This library include tests, just run `python runtests.py`

You can also run separate test cases: `runtests.py tests.test_filters.ReactIncludeComponentTest`


## Contributing

Want to contribute? Awesome. Just send a pull request.


## License

Django-React-Templatetags is released under the [MIT License](http://www.opensource.org/licenses/MIT).
