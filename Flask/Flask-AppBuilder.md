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

# appendix


##  define a FilterInFunction for one-to-many relation in flask appbuilder

To define a `FilterInFunction` for a one-to-many relationship in Flask AppBuilder, you can create a custom filter function that returns the appropriate values for filtering. Here’s a step-by-step guide:

1. **Define Your Models**: Ensure you have a one-to-many relationship set up in your models.
    
2. **Create the Custom Filter Function**: Define a function that returns the values you want to filter by.
    
3. **Apply the Filter in Your View**: Use `FilterInFunction` in your view to apply the custom filter.
    

Here’s an example:

### Step 1: Define Your Models

```python
from flask_appbuilder import Model
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

class ParentModel(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    children = relationship("ChildModel", back_populates="parent")

class ChildModel(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    parent_id = Column(Integer, ForeignKey('parent_model.id'), nullable=False)
    parent = relationship("ParentModel", back_populates="children")
```

`roles = relationship("Role", secondary=assoc_user_role, backref="user")`
### Step 2: Create the Custom Filter Function

```python
def get_child_ids():
    # Example function to get child IDs based on some logic
    return [child.id for child in ChildModel.query.all()]
```

### Step 3: Apply the Filter in Your View

```python
from flask_appbuilder import ModelView, SQLAInterface
from flask_appbuilder.models.sqla.filters import FilterInFunction

class ParentModelView(ModelView):
    datamodel = SQLAInterface(ParentModel)
    
    # Apply the custom filter using FilterInFunction
    base_filters = [['children', FilterInFunction, get_child_ids]]

appbuilder.add_view(
    ParentModelView,
    "Parent Model",
    icon="fa-folder-open-o",
    category="Your Category"
)
```

In this example:

- **Models**: `ParentModel` and `ChildModel` are set up with a one-to-many relationship.
- **Custom Filter Function**: `get_child_ids` returns a list of child IDs.
- **View**: `ParentModelView` uses `FilterInFunction` to filter `ParentModel` based on the child IDs returned by `get_child_ids`.

[This approach allows you to create dynamic filters for one-to-many relationships in your Flask AppBuilder application](https://www.digitalocean.com/community/tutorials/how-to-use-one-to-many-database-relationships-with-flask-sqlalchemy)[1](https://www.digitalocean.com/community/tutorials/how-to-use-one-to-many-database-relationships-with-flask-sqlalchemy).

If you have any specific requirements or run into any issues, feel free to ask!

## **OR** logic for search filters in


In **Flask AppBuilder** (FAB), applying **OR** logic for search filters in the UI can be a bit tricky, as the default behavior is to use **AND** between multiple filters. However, you can accomplish **OR** filtering in the UI search by using **custom filters** and modifying the filter logic. Here's how you can do it:

### Solution: Custom Filters with OR Logic

1. **Create Custom Filters**: Define custom filter classes for each of the fields you want to combine with OR logic. In the `apply` method, include the OR condition using SQLAlchemy expressions.
  
2. **Combine Filters in a Single UI Search Field**: Use a combined filter approach to include multiple fields in a single custom filter, so users can apply OR conditions directly in the UI.

### Example: OR Filter on `first_name` and `last_name` Fields

Let's say you have an `Employee` model, and you want to enable an OR search on the `first_name` and `last_name` fields. 

#### Step 1: Define the Custom OR Filter

Define a custom filter class that applies an OR condition between the `first_name` and `last_name` fields.

```python
from flask_appbuilder import ModelView, BaseFilter, SQLAInterface
from sqlalchemy import or_
from .models import Employee

class FilterFirstOrLastNameContains(BaseFilter):
    name = "First or Last Name Contains"  # Display name for the filter in the UI

    def apply(self, query, value):
        # Apply OR condition between first_name and last_name fields
        return query.filter(
            or_(
                Employee.first_name.ilike(f"%{value}%"),
                Employee.last_name.ilike(f"%{value}%")
            )
        )
```

#### Step 2: Add the Custom Filter to Your ModelView

Now, integrate this custom filter into your `ModelView` so it appears in the UI search options.

```python
class EmployeeView(ModelView):
    datamodel = SQLAInterface(Employee)

    # Adding custom filter to the search filters
    search_columns = ['first_or_last_name']

    # Adding the custom filter directly to the filter attributes
    base_filters = [
        ['first_or_last_name', FilterFirstOrLastNameContains, '']
    ]

# Register the view
appbuilder.add_view(EmployeeView, "Employees", category="Management")
```

#### Step 3: Enable UI Access to the Custom Filter

With the `base_filters` and `search_columns` settings, your new filter, "First or Last Name Contains," will be available in the FAB UI, and it will use **OR** logic when filtering `first_name` or `last_name` based on the search term.

### Important Points

- The custom filter name (`name`) will appear in the search options in the UI, helping users understand what the filter does.
- If you want to add more fields to the OR logic, simply extend the `or_()` clause in the custom filter’s `apply` method.
- You can add more custom filters to `search_columns` for additional filtering options with OR logic as needed.

This approach provides a user-friendly way to filter with **OR** conditions directly in the FAB UI.


## Display a hyperlink field in Flask AppBuilder 


### Custom Fields[¶](#custom-fields "Link to this heading")
-------------------------------------------------------

Custom Model properties can be used on lists. This is useful for formatting values like currencies, time or dates. or for custom HTML. This is very simple to do, first define your custom property on your Model and then use the **@renders** decorator to tell the framework to map you class method with a certain Model property:

from flask\_appbuilder.models.decorators import renders

class MyModel(Model):
    id \= Column(Integer, primary\_key\=True)
    name \= Column(String(50), unique \= True, nullable\=False)
    custom \= Column(Integer(20))

    @renders('custom')
    def my_custom(self):
     # will render this columns as bold on ListWidget
        return Markup('<b>' + self.custom + '</b>')

On your view reference your method as a column on list:

class MyModelView(ModelView):
    datamodel \= SQLAInterface(MyTable)
    list\_columns \= \['name', 'my\_custom'\]


## hide the “Delete Record” button in Flask AppBuilder

To hide the “Delete Record” button in Flask AppBuilder, you can customize the permissions and actions for your views. Here are a few steps you can follow:

1. **Modify Permissions**: Ensure that the delete permission is not assigned to the roles that should not see the delete button.
    
2. **Customize Actions**: You can override the default actions in your view to hide the delete button. Here’s an example of how to do this:
    
    ```python
    from flask_appbuilder import ModelView
    from flask_appbuilder.models.sqla.interface import SQLAInterface
    from your_application import appbuilder, db
    from your_application.models import YourModel
    
    class YourModelView(ModelView):
        datamodel = SQLAInterface(YourModel)
        
        # Override the default actions
        list_columns = ['column1', 'column2']  # Specify the columns you want to display
        show_columns = ['column1', 'column2']
        edit_columns = ['column1', 'column2']
        add_columns = ['column1', 'column2']
    
        # Remove the delete action
        base_permissions = ['can_list', 'can_show', 'can_add', 'can_edit']
    
    appbuilder.add_view(
        YourModelView,
        "Your Model",
        icon="fa-folder-open-o",
        category="Your Category"
    )
    ```
    
3. **Use the `@action` Decorator**: If you have custom actions, you can control their visibility using the `@action` decorator. For example, to disable the delete action:
    
    ```python
    from flask_appbuilder import ModelView
    from flask_appbuilder.models.sqla.interface import SQLAInterface
    from your_application import appbuilder, db
    from your_application.models import YourModel
    
    class YourModelView(ModelView):
        datamodel = SQLAInterface(YourModel)
    
        @action("muldelete", "Delete", "Delete all Really?", "fa-rocket", single=False)
        def muldelete(self, items):
            # Custom delete logic
            pass
    
        # Disable the delete action
        actions_exclude = ['muldelete']
    
    appbuilder.add_view(
        YourModelView,
        "Your Model",
        icon="fa-folder-open-o",
        category="Your Category"
    )
    ```
    

[By following these steps, you can effectively hide the “Delete Record” button in your Flask AppBuilder application](https://flask-appbuilder.readthedocs.io/en/latest/actions.html)[1](https://flask-appbuilder.readthedocs.io/en/latest/actions.html).

## relation
In Flask AppBuilder, the `relation()` function is used to define relationships between models. This function is part of SQLAlchemy, which Flask AppBuilder uses for ORM (Object-Relational Mapping). Here are the key parameters you can use with `relation()`:

1. **Argument**: The class or string name of the related model.
2. **back_populates**: Specifies the complementary property on the related model.
3. **backref**: A shortcut to create a bidirectional relationship.
4. **cascade**: Defines how related objects should be treated when the parent object is acted upon.
5. **collection_class**: Specifies a custom collection class for the relationship.
6. **comparator_factory**: A custom comparator factory for the relationship.
7. **doc**: Documentation string for the relationship.
8. **extension**: Deprecated; use `comparator_factory` instead.
9. **foreign_keys**: Specifies the foreign key columns involved in the relationship.
10. **info**: A dictionary of arbitrary user-defined information.
11. **innerjoin**: When set to `True`, SQLAlchemy will use an inner join instead of an outer join.
12. **join_depth**: Limits the depth of eager loading for the relationship.
13. **lazy**: Defines how the related items should be loaded.
14. **load_on_pending**: If `True`, the relationship will be loaded even if the parent object is pending.
15. **order_by**: Specifies the ordering of the related items.
16. **passive_deletes**: Controls how deletions are handled.
17. **passive_updates**: Controls how updates are handled.
18. **post_update**: Allows the relationship to be updated after the parent object.
19. **primaryjoin**: Specifies the primary join condition for the relationship.
20. **remote_side**: Specifies the remote side of the relationship.
21. **secondary**: Specifies the secondary table for many-to-many relationships.
22. **secondaryjoin**: Specifies the join condition for the secondary table.
23. **single_parent**: Ensures that a child object can only be associated with one parent at a time.
24. **uselist**: When set to `False`, the relationship will be a scalar value instead of a list.
25. **viewonly**: When set to `True`, the relationship will be read-only.

Here’s an example of how you might define a relationship using some of these parameters:

```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship
from flask_appbuilder import Model

class ParentModel(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    children = relationship("ChildModel", back_populates="parent")

class ChildModel(Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    parent_id = Column(Integer, ForeignKey('parent_model.id'), nullable=False)
    parent = relationship("ParentModel", back_populates="children")
```

In this example:

- `children` in `ParentModel` and `parent` in `ChildModel` use the `back_populates` parameter to create a bidirectional relationship.

[For more detailed information, you can refer to the SQLAlchemy documentation and the](https://flask-appbuilder.readthedocs.io/en/latest/relations.html) [1](https://flask-appbuilder.readthedocs.io/en/latest/relations.html).

## in flask appbuilder GroupByChartView, how to handle the fields have NoneType object

In **Flask AppBuilder (FAB)**, if you want to set a default value for a numeric field in a `GroupByChartView`, you can handle `None` values in your data by using a custom query or by applying a function like `coalesce` to set a default value. This approach ensures that any `None` values are replaced with a default before the data is visualized in the chart.

Here’s how to set a default value for a numeric field in a `GroupByChartView`:

### 1. Use `func.coalesce` to Set a Default Value in the Chart Query

You can use SQLAlchemy’s `func.coalesce` function to replace `None` values with a default numeric value (e.g., `0`) directly in the query.

#### Example

Suppose you have an `Employee` model with a numeric field `salary`, which may contain `None` values. To set a default value (like `0`) for `salary` in the chart, you can apply `func.coalesce` in the `base_query`.

```python
from flask_appbuilder import GroupByChartView
from flask_appbuilder.models.group import AggregateSum
from sqlalchemy import func
from .models import Employee

class EmployeeChartView(GroupByChartView):
    datamodel = SQLAInterface(Employee)
    chart_title = 'Total Salary by Department'
    
    # Define base query to replace None with 0 for salary
    base_query = (
        lambda self: self.datamodel.session.query(
            Employee.department,
            func.coalesce(Employee.salary, 0).label("salary")
        )
    )

    definitions = [
        {
            "label": "Department",
            "group": "department",
            "series": [
                (AggregateSum, "salary")
            ]
        }
    ]
```

In this example:
- `func.coalesce(Employee.salary, 0)` replaces `None` values in `salary` with `0`.
- The `salary` field is then labeled for use in the chart aggregation.

### 2. Use `label_columns` with `func.coalesce` (Alternative)

If you’re working with a single numeric field and need a simpler solution, you can define a `label_columns` attribute with `func.coalesce` to format the field, though `base_query` is generally more flexible.

```python
class EmployeeChartView(GroupByChartView):
    datamodel = SQLAInterface(Employee)
    chart_title = 'Total Salary by Department'
    
    # Replace None values in `salary` with 0
    label_columns = {
        'salary': func.coalesce(Employee.salary, 0)
    }

    definitions = [
        {
            "label": "Department",
            "group": "department",
            "series": [
                (AggregateSum, "salary")
            ]
        }
    ]
```

In this setup, `func.coalesce(Employee.salary, 0)` will replace `None` values with `0` when displaying the `salary` field in the chart.

### Summary

Using `func.coalesce` in `base_query` or `label_columns` ensures that `None` values are replaced with a specified default value (`0` in this example), preventing errors and displaying consistent data in your charts. Choose the method that best suits your chart requirements.