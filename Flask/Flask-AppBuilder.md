# Views

Each view is a Flask **blueprint** that will be created for you automatically by the framework. You will map your methods to routing points, and each method will be registered as a possible security permission if you want. 

## BaseView

All views inherit from this class. Its constructor will register your exposed urls on flask as a **Blueprint**, as well as all security permissions that need to be defined and protected.

You can use this kind of view to implement your own custom pages, attach it to a menu or link it from any point to your site.

Decorate your url routing methods with **@expose**. Additionally add **@has_access** decorator to tell flask that this is a security protected method.

```
from flask_appbuilder import AppBuilder, BaseView, expose, has_access
from app import appbuilder

class MyView(BaseView):

    default_view = 'method1'

    @expose('/method1/')
    @has_access
    def method1(self):
        # do something with param1
        # and return to previous page or index
        return 'Hello'

    @expose('/method2/<string:param1>')
    @has_access
    def method2(self, param1):
        # do something with param1
        # and render template with param
        param1 = 'Goodbye %s' % (param1)
        return param1

appbuilder.add_view(MyView, "Method1", category='My View')
appbuilder.add_link("Method2", href='/myview/method2/john', category='My View')
```

## Form views

Subclass **SimpleFormView** or **PublicFormView** to provide base processing for your customized form views.

Usually you will need this kind of view to present forms,  F.A.B. can automatically generate them and you can add or remove fields to it, as well as custom validators. To create a custom form view, first define your [WTForm](https://wtforms.readthedocs.org/en/latest/) fields and inherit them from F.A.B. **_DynamicForm_**.

MyForm:

```
from wtforms import Form, StringField
from wtforms.validators import DataRequired
from flask_appbuilder.fieldwidgets import BS3TextFieldWidget
from flask_appbuilder.forms import DynamicForm

class MyForm(DynamicForm):
    field1 = StringField(('Field1'),
        description=('Your field number one!'),
        validators = [DataRequired()], widget=BS3TextFieldWidget())
    field2 = StringField(('Field2'),
        description=('Your field number two!'), widget=BS3TextFieldWidget())
```

Now define your form view to expose urls, create a menu entry, create security accesses, define pre and post processing.

Implement **_form_get_** and **_form_post_** to implement your form pre-processing and post-processing. You can use **_form_get_** to prefill the form with your data, and/or pre process something on your application, then use **_form_post_** to post process the form after the user submits it, you can save the data to database, send an email or any other action required.

On your **form_post** method, you can also return None, or a Flask response to render a custom template or redirect the user.

MyFromView:

```
from flask import flash
from flask_appbuilder import SimpleFormView
from flask_babel import lazy_gettext as _

class MyFormView(SimpleFormView):
    form = MyForm
    form_title = 'This is my first form view'
    message = 'My form submitted'

    def form_get(self, form):
        form.field1.data = 'This was prefilled'

    def form_post(self, form):
        # post process form
        flash(self.message, 'info')

appbuilder.add_view(MyFormView, "My form View", icon="fa-group", label=_('My form View'),
                     category="My Forms", category_icon="fa-cogs")
```

Most important Base Properties:

| form_title:   | The title to be presented (this is mandatory)                                              |
|:------------- |:------------------------------------------------------------------------------------------ |
| form_columns: | The form column names to include                                                           |
| form:         | Your form class ([WTForm](https://wtforms.readthedocs.org/en/latest/)) (this is mandatory) |

## ModelView on SQLAlchemy

Inheriting your model classes from **Model** class is recommended. **Model** class is exactly the same as Flask-SQLALchemy **db.Model** but without the underlying connection. You can of course inherit from **db.Model** normal Flask-SQLAlchemy. The reason for this is that **Model** is on the same declarative space of F.A.B. and using it will allow you to define relations to Users.

You can add automatic _Audit_ triggered columns to your models, by inheriting them from _AuditMixin_ also. (see [API Reference](https://flask-appbuilder.readthedocs.io/en/latest/api.html))


## ModelView on MongoDB


### Custom Fields[¶](https://flask-appbuilder.readthedocs.io/en/latest/advanced.html#custom-fields "Link to this heading")

Custom Model properties can be used on lists. This is useful for formatting values like currencies, time or dates. or for custom HTML. This is very simple to do, first define your custom property on your Model and then use the **@renders** decorator to tell the framework to map you class method with a certain Model property:
```
from flask_appbuilder.models.decorators import renders

class MyModel(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique = True, nullable=False)
    custom = Column(Integer(20))

    @renders('custom')
    def my_custom(self):
    # will render this columns as bold on ListWidget
        return Markup('<b>' + self.custom + '</b>')

```

On your view reference your method as a column on list:
```
class MyModelView(ModelView):
    datamodel = SQLAInterface(MyTable)
    list_columns = ['name', 'my_custom']
```
