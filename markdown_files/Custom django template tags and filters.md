# Custom Django tags and filters

Custom tags in Django are a powerful feature that allows you to extend the template language with your own custom functionality. This can be particularly useful when you need to encapsulate complex logic or render specific output that is reused across your templates.

Here’s a step-by-step guide on how to create and use custom template tags in a Django application:

### Step 1: Create a `templatetags` Directory

1. Create a `templatetags` Directory,
>In your Django app, create a directory named templatetags. This directory is where you’ll define your custom tags and filters. The `templatetags` directory must contain an `__init__.py` file to make it a Python package.

```
myapp/
    templatetags/
        __init__.py
        custom_tags.py
```

2. Ensure Your App is in `INSTALLED_APPS`:

>Make sure your app is listed in the INSTALLED_APPS setting in your Django project’s settings.py file.

```python
INSTALLED_APPS = [
    # ...
    'myapp',
    # ...
]
```

### Step 2: Define a Custom Tag or Filter

>In your `templatetags/custom_tags.py` file, you can define your custom tags or filters.

#### Example: Custom Template Tag

```python
# myapp/templatetags/custom_tags.py

from django import template

register = template.Library()

@register.simple_tag
def capitalize_words(value):
    """Capitalizes the first letter of each word in a string."""
    return value.title()
```

#### Example: Custom Template Filter

```python
# myapp/templatetags/custom_tags.py

from django import template

register = template.Library()

@register.filter(name='upper')
def upper(value):
    """Converts a string to uppercase."""
    return value.upper()
```

### Step 3: Load Custom Tags in Templates
Once you’ve defined your custom tags or filters, you need to load them in your templates before using them.

#### Loading Custom Tags
>In your template file (e.g., `template.html`), load the custom tags using the {% load %} tag:

```django
{% load custom_tags %}

<h1>{% capitalize_words "hello world" %}</h1> <!-- Outputs: Hello World -->
```

#### Using Custom Filters
>To use the custom filter in a template, use the pipe `|` syntax:

```django
{% load custom_tags %}

<p>{{ "hello world" | upper }}</p> <!-- Outputs: HELLO WORLD -->
```

### Step 4: Advanced Custom Tags
Django also supports more advanced custom tags that can take multiple arguments, render custom HTML, or even execute complex logic. Here’s an example of using a custom tag with arguments and rendering HTML content.

#### Example: Custom Tag with Arguments

```python
# myapp/templatetags/custom_tags.py

from django import template

register = template.Library()

@register.simple_tag
def greet(name, greeting="Hello"):
    """Returns a greeting with the provided name."""
    return f"{greeting}, {name}!"
```

Usage in a template:

```django
{% load custom_tags %}

<p>{% greet "Alice" %}</p> <!-- Outputs: Hello, Alice! -->
<p>{% greet "Alice" greeting="Hi" %}</p> <!-- Outputs: Hi, Alice! -->
```

#### Example: Custom Inclusion Tag
>Inclusion tags render a specific template with the provided context, which is useful for rendering reusable HTML components

```python
# myapp/templatetags/custom_tags.py

from django import template

register = template.Library()

@register.inclusion_tag('myapp/user_profile.html')
def user_profile(user):
    """Renders a user profile with the given user object."""
    return {'user': user}
```

`user_profile.html` template:

```django
<!-- myapp/templates/myapp/user_profile.html -->

<div class="user-profile">
    <h3>{{ user.name }}</h3>
    <p>Email: {{ user.email }}</p>
</div>
```

Usage in a template:

```django
{% load custom_tags %}

{% user_profile request.user %}
```

### Step 5: Handling Complex Logic
For more complex logic, you can create custom template tags using the `template.Node` and `template.Parser` classes, allowing you to perform complex operations and manipulate the template rendering process directly.

#### Example: Custom Node Tag

```python
# myapp/templatetags/custom_tags.py

from django import template

register = template.Library()

class RepeatNode(template.Node):
    def __init__(self, times, nodelist):
        self.times = template.Variable(times)
        self.nodelist = nodelist

    def render(self, context):
        times = self.times.resolve(context)
        output = ''
        for _ in range(int(times)):
            output += self.nodelist.render(context)
        return output

@register.tag(name='repeat')
def do_repeat(parser, token):
    try:
        tag_name, times = token.split_contents()
    except ValueError:
        raise template.TemplateSyntaxError("Tag 'repeat' requires a single argument")
    nodelist = parser.parse(('endrepeat',))
    parser.delete_first_token()
    return RepeatNode(times, nodelist)
```

Usage in a template:

```django
{% load custom_tags %}

{% repeat 3 %}
    <p>This will repeat three times.</p>
{% endrepeat %}
```

### Step 6: Testing and Debugging
When developing custom tags and filters, it's crucial to test them thoroughly. You can add unit tests to ensure they work as expected.

#### Example: Testing Custom Template Tags
You can create test cases for your custom tags using Django's `TestCase`.

```python
# myapp/tests/test_custom_tags.py

from django.test import TestCase
from django.template import Template, Context

class CustomTagsTestCase(TestCase):
    def test_capitalize_words(self):
        template = Template('{% load custom_tags %}{{ "hello world"|capitalize_words }}')
        rendered = template.render(Context())
        self.assertEqual(rendered, 'Hello World')
    
    def test_upper_filter(self):
        template = Template('{% load custom_tags %}{{ "hello world"|upper }}')
        rendered = template.render(Context())
        self.assertEqual(rendered, 'HELLO WORLD')
```

### Conclusion
>Custom tags and filters in Django are an excellent way to extend the template system and encapsulate reusable functionality. By following the steps outlined above, you can create powerful custom tags and filters tailored to your application’s needs.

