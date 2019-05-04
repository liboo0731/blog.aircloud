---
title: 【python】htmlpy框架使用
date: 2017-11-25 11:23:49
tags:
- python
- htmlpy
---

### htmlPy Project

```
back_end_codes/
static/
    css/
        style.css
        .
        .
        .
    js/
        script.js
        .
        .
        .
    img/
        logo.img
        .
        .
        .
templates/
    first_template_directory/
        template1.html
        template2.html
        .
        .
    another_template_directory/
        another_template.html
    base_layout.html
main.py
```

### Standalone application

```python
import htmlPy
import os

app = htmlPy.AppGUI(title=u"htmlPy Quickstart", maximized=True)
app.template_path = os.path.abspath(".")
app.static_path = os.path.abspath(".")
app.template = ("index.html", {"username": "htmlPy_user"})

app.start()
```

### Web based application

```python
import htmlPy


web_app = htmlPy.WebAppGUI(title=u"Python Website", maximized=True)
web_app.url = u"<http://python.org/>"

web_app.start()
```

### Here’s a sample driver file

```python
import os
import htmlPy
from PyQt4 import QtGui

# Initial confiurations
BASE_DIR = os.path.abspath(os.path.dirname(__file__))
# GUI initializations
app = htmlPy.AppGUI(title=u"Application", maximized=True, plugins=True)

# GUI configurations
app.static_path = os.path.join(BASE_DIR, "static/")
app.template_path = os.path.join(BASE_DIR, "templates/")

app.web_app.setMinimumWidth(1024)
app.web_app.setMinimumHeight(768)
app.window.setWindowIcon(QtGui.QIcon(BASE_DIR + "/static/img/icon.png"))

# Binding of back-end functionalities with GUI

# Import back-end functionalities
from html_to_python import ClassName

# Register back-end functionalities
app.bind(ClassName())

# Instructions for running application

if __name__ == "__main__":
    # The driver file will have to be imported everywhere in back-end.
    # So, always keep app.start() in if __name__ == "__main__" conditional

    app.start()
```

### Set static_path and template_path

```html
<script src="{{ 'js/jquery.min.js'|staticfile }}"></script>
<link rel="stylesheet" href="{{ 'css/bootstrap.min.css'|staticfile }}">
```

### GUI to Python calls

```python
import htmlPy
import json
from sample_app import app as htmlPy_app


class ClassName(htmlPy.Object):
    # GUI callable functions have to be inside a class.
    # The class should be inherited from htmlPy.Object.

    def __init__(self):
        super(ClassName. self).__init__()
        # Initialize the class here, if required.
        return

    @htmlPy.Slot()
    def function_name(self):
        # This is the function exposed to GUI events.
        # You can change app HTML from here.
        # Or, you can do pretty much any python from here.
        #
        # NOTE: @htmlPy.Slot decorater needs argument and return data-types.
        # Refer to API documentation.
        return

    @htmlPy.Slot(str, result=str)
    def form_function_name(self, json_data):
        # @htmlPy.Slot(arg1_type, arg2_type, ..., result=return_type)
        # This function can be used for GUI forms.
        #
        form_data = json.loads(json_data)
        return json.dumps(form_data)

    @htmlPy.Slot()
    def javascript_function(self):
        # Any function decorated with @htmlPy.Slot decorater can be called
        # using javascript in GUI
        return
```



```html
<a href="ClassName.function_name" id="link" data-bind="true">GUI clickable link</a>
<!-- The "a" tag needs to have unique ID and data-bind attribute set to "true"
The ClassName and function_name have to be set in href attribute as displayed above.
The "a" tag can be styled using CSS with other HTML elements inside it -->

<form action="ClassName.form_function_name" id="form" data-bind="true">
    <input type="text" id="form_input" name="name">
    <input type="submit" value="Submit" id="form_submit">
</form>
<!-- The "form" tag needs to have unique ID and data-bind attribute set to "true".
The ClassName and form_function_name have to be set in action attribute as
displayed above. The argument given to ClassName.form_function_name on form submit
will be a json string of the form data. -->

<script>
ClassName.javascript_function();
// You can treat the class inherited from htmlPy.Object (in this case, ClassName)
// as a javascript object.
</script>
```

#### Python to GUI calls

```python
from sample_app import app

# app imported from sample_app file is an instance of htmlPy.AppGUI class.
# Change HTML of the app
app.html = u"<html></html>"

# Change HTML of the app using Jinja2 templates
app.template = ("./index.html", {"template_variable_name": "value"})

# Execute javascript on currently displayed HTML in the app
app.evaluate_javascript("alert('Hello from back-end')")
```

### Using file input

```html
<input type="file" name="file" id="file" data-filter="[{'title': 'Images', 'extensions': '*.png *.xpm *.jpg'}, {'title': 'Documents', 'extensions': '*.pdf *.doc *.docx'}]">
```

### Integration with django

```python
import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "<project_name>.settings")
```

