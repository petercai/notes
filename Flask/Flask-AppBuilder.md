
# How can I use Flask-AppBuilder's built-in features to conditionally show or hide buttons

To conditionally show or hide buttons in Flask-AppBuilder, you can leverage its built-in features and customize your views. Here’s how to do it effectively:

## Method 1: Using `menu_cond` for Conditional Menu Items

Flask-AppBuilder allows you to conditionally display menu items using the `menu_cond` parameter when adding views. This can be used to control the visibility of action buttons based on specific conditions.

### Example Implementation

1. **Define a Condition Function**:
   Create a function that returns a boolean value based on your conditions.

   ```python
   def is_feature_enabled(feature_name):
       # Replace this with your actual logic
       return feature_name in enabled_features_list
   ```

2. **Add the View with Conditional Menu**:
   Use the `menu_cond` parameter when adding your view.

   ```python
   from flask_appbuilder import AppBuilder

   appbuilder.add_view(
       MyModelView,
       "My View",
       icon='fa-eye',
       menu_cond=lambda: is_feature_enabled("my-feature")
   )
   ```

This setup ensures that the view (and its associated buttons) will only be visible if the condition returns `True`.

## Method 2: Overriding Methods in Your View Class

You can also override methods in your view class to control button visibility dynamically based on item attributes.

### Example Implementation

1. **Override `get_list` Method**:
   Modify the `get_list` method to add attributes that indicate whether buttons should be shown.

   ```python
   class MyModelView(ModelView):
       datamodel = SQLAInterface(MyModel)

       def get_list(self, *args, **kwargs):
           items = super().get_list(*args, **kwargs)
           for item in items:
               item.show_action_button = (item.type == 'active')  # Example condition
           return items
   ```

2. **Update Your Template**:
   In your Jinja template, check the new attribute to conditionally render buttons.

   ```html
   {% for item in items %}
       <tr>
           <td>{{ item.name }}</td>
           {% if item.show_action_button %}
               <td><a href="{{ url_for('MyModelView.my_action', id=item.id) }}">Perform Action</a></td>
           {% else %}
               <td><span class="disabled">Action Unavailable</span></td>
           {% endif %}
       </tr>
   {% endfor %}
   ```

## Method 3: Using Built-in Properties

Flask-AppBuilder provides built-in properties like `can_add`, `can_edit`, etc., which can be set to control button visibility directly.

```python
class MyModelView(ModelView):
    can_add = False  # Disable add button based on conditions
    can_edit = True  # Enable edit button based on conditions
```

## Summary

By utilizing these methods, you can effectively control the visibility of action buttons in Flask-AppBuilder based on various conditions. Whether through conditional menu items, overriding view methods, or using built-in properties, Flask-AppBuilder offers flexibility to tailor the UI according to your application's needs.

Citations:
[1] https://flask-appbuilder.readthedocs.io/en/latest/api.html
[2] https://discourse.psychopy.org/t/conditionally-hide-a-button-component/32324
[3] https://stackoverflow.com/questions/65024365/handling-button-visibility-in-flask-with-jinja
[4] https://flask-appbuilder.readthedocs.io/en/latest/customizing.html
[5] https://github.com/dpgaspar/Flask-AppBuilder/issues/752
[6] https://stackoverflow.com/questions/tagged/flask-appbuilder
[7] https://realpython.com/flask-javascript-frontend-for-rest-api/
[8] https://flask.palletsprojects.com/en/stable/changes/

# 1 generate recored id in url

In Flask-Appbuilder, when viewing a specific record in the app, the record's ID is often included in the URL as part of the route. Flask-Appbuilder uses dynamic routing to show a particular record.

For example, when viewing an individual record in a model, the URL typically looks something like this:

```
/<model_name>/<record_id>/
```

Here’s a breakdown of how Flask-Appbuilder handles the record ID in the URL:

1. **Model-Based Routing**: Each model in Flask-Appbuilder has a corresponding route in the application that serves the details of a specific record.
    
2. **Route URL Format**: Flask-Appbuilder dynamically generates the URL for a record based on its ID. For example, if you have a model called `User` and the ID of the record is `5`, the URL might look like:
    
    ```
    /user/view/5/
    ```
    
    Here:
    
    - `/user/`: This part of the URL identifies the model (`User` in this case).
    - `/view/`: This indicates the view endpoint for showing a record.
    - `5/`: This is the ID of the record being viewed.
3. **Controller Action**: Flask-Appbuilder associates views and actions in controllers. When you define a view for a model, you typically have a route that will handle displaying a specific record based on its ID. This is done through Flask's routing and Flask-Appbuilder’s view system.
    
    Here's an example of how a `view` route might be defined for a model in Flask-Appbuilder:
    
    ```python
    from flask_appbuilder import ModelView
    
    class UserModelView(ModelView):
        # other configurations
    
        @expose("/view/<int:id>/")
        def view(self, id):
            user = self.session.query(User).get(id)  # Fetch the user by ID
            return self.render_template('user_view.html', user=user)
    ```
    
    In this case:
    
    - The `<int:id>` part in the route dynamically captures the ID of the record from the URL.
    - The record with the corresponding ID is then fetched from the database and passed to the template for rendering.
4. **URL Generation in AppBuilder**: When you use Flask-Appbuilder's built-in views, it automatically generates these routes based on the model and view configuration. If you want to generate a URL for a specific record, you can use the `url_for` function, passing the record's ID to it.
    

For example, generating a link to a user with ID `5` might look like:

```python
url_for('UserModelView.view', id=5)
```

This will produce the URL like `/user/view/5/`.

### Summary:

- Flask-Appbuilder generates the record ID in the URL dynamically as part of the routing system.
- The URL typically follows the pattern `/<model_name>/view/<record_id>/` for viewing records.
- Flask’s dynamic URL pattern (e.g., `<int:id>`) is used to capture the record's ID and display the corresponding data in the view.

# use primary key as url id

Yes, in Flask-Appbuilder, the `id` in the URL typically corresponds to the **primary key** of the table in the database.

Flask-Appbuilder automatically assumes that the model's primary key (often called `id`) is used for record identification in routes, unless you explicitly configure it otherwise.

### Key points:

1. **Primary Key by Default**: By default, the `id` field in your model is treated as the primary key. This is usually an auto-incrementing integer field in the database that uniquely identifies each record.
    
2. **URL Structure**: When Flask-Appbuilder generates URLs for viewing records, the `id` in the URL represents this primary key. For example, for a model `User`, a route like `/user/view/5/` implies that the `id` of the user being viewed is `5`, and this `id` is the primary key for that particular record.
    
3. **Database Table**: Flask-Appbuilder assumes that the model class will have a column, typically named `id`, that is marked as the primary key. For example:
    
    ```python
    from flask_appbuilder import Model
    from sqlalchemy import Column, Integer, String
    
    class User(Model):
        id = Column(Integer, primary_key=True)  # This is the primary key
        name = Column(String(100))
    ```
    
4. **Custom Primary Key**: If you use a custom primary key field, such as `user_id`, you would need to adjust the logic accordingly. For instance, you would need to define how the routing should capture and use that custom field instead of the default `id`.
    

### Example with Custom Primary Key:

If you have a model where the primary key is not `id`, you can configure it as follows:

```python
from flask_appbuilder import Model
from sqlalchemy import Column, Integer, String

class User(Model):
    user_id = Column(Integer, primary_key=True)  # Custom primary key field
    name = Column(String(100))
```

In that case, you would adjust your route to capture the custom primary key:

```python
class UserModelView(ModelView):
    @expose("/view/<int:user_id>/")
    def view(self, user_id):
        user = self.session.query(User).get(user_id)  # Fetch using the custom user_id
        return self.render_template('user_view.html', user=user)
```

In the URL, this would look like `/user/view/5/`, where `5` is the value of the `user_id` (the primary key).

### Summary:

- **Yes**, the `id` in the URL typically refers to the primary key of the table (e.g., `id` field in the model).
- If you are using the default model setup, Flask-Appbuilder assumes `id` is the primary key.
- If you use a custom primary key, you'll need to adjust your routes and logic to reflect that change.

# composite key

If your table in Flask-Appbuilder has a **composite key** (a primary key made up of two or more columns), the default behavior of Flask-Appbuilder, which assumes a single `id` field as the primary key, needs to be adjusted.

For a composite key, you must explicitly handle how Flask-Appbuilder generates URLs and how records are accessed. Flask-Appbuilder does not natively support composite keys directly in URLs, but you can work around this by using a custom approach.

Here’s how you can control the URL generation and handling for a composite key.

### Steps to Handle Composite Keys in Flask-Appbuilder:

#### 1. Define the Composite Key

You need to define your model with the composite primary key using SQLAlchemy.

For example, if you have a model with a composite key made up of `key_part1` and `key_part2`, you would define it as follows:

```python
from flask_appbuilder import Model
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import validates

class MyModel(Model):
    key_part1 = Column(Integer, primary_key=True)  # First part of the composite key
    key_part2 = Column(String(50), primary_key=True)  # Second part of the composite key
    name = Column(String(100))

    def __init__(self, key_part1, key_part2, name):
        self.key_part1 = key_part1
        self.key_part2 = key_part2
        self.name = name
```

#### 2. Create a Custom URL Route with Composite Key

Since Flask-Appbuilder expects a single primary key (`id`), you need to adjust your view route to accept both parts of the composite key.

For example, you can define a view for your model like this:

```python
from flask_appbuilder import ModelView
from flask_appbuilder.charts.views import ModelChartView
from flask_appbuilder import expose

class MyModelView(ModelView):
    datamodel = SQLAInterface(MyModel)

    @expose('/view/<int:key_part1>/<string:key_part2>/')
    def view(self, key_part1, key_part2):
        record = self.datamodel.session.query(MyModel).filter_by(key_part1=key_part1, key_part2=key_part2).first()
        if record:
            return self.render_template('my_model_view.html', record=record)
        else:
            return self.render_template('404.html')  # Handle record not found
```

In this example:

- The URL is `/view/<int:key_part1>/<string:key_part2>/`, where `key_part1` and `key_part2` correspond to the parts of your composite primary key.
- You use the composite key values to query the database and retrieve the record.

#### 3. Generating URLs

If you want to generate URLs for a record with a composite key, you need to use `url_for` and pass both key parts:

```python
url_for('MyModelView.view', key_part1=123, key_part2="abc")
```

This would generate a URL like `/my_model/view/123/abc/`.

#### 4. Handling the Composite Key in the Template

In your template, you can access the `key_part1` and `key_part2` fields just as you would any other fields from the model.

```html
<h1>{{ record.name }}</h1>
<p>Key Part 1: {{ record.key_part1 }}</p>
<p>Key Part 2: {{ record.key_part2 }}</p>
```

#### 5. Update the `list` and `edit` Views (Optional)

You can similarly update the list or edit views to handle composite keys. You may need to modify the URLs and queries accordingly. For example:

```python
class MyModelView(ModelView):
    datamodel = SQLAInterface(MyModel)

    list_columns = ['key_part1', 'key_part2', 'name']

    @expose('/edit/<int:key_part1>/<string:key_part2>/')
    def edit(self, key_part1, key_part2):
        record = self.datamodel.session.query(MyModel).filter_by(key_part1=key_part1, key_part2=key_part2).first()
        if record:
            return self.render_template('my_model_edit.html', record=record)
        else:
            return self.render_template('404.html')
```

### Summary of Key Steps:

- **Model**: Define your composite primary key in the model using multiple columns marked as `primary_key=True`.
- **Route**: Customize the route for views (`view`, `edit`, etc.) to accept both parts of the composite key.
- **Query**: Adjust the query logic to filter by both parts of the composite key.
- **URL Generation**: Use `url_for` with both key parts to generate URLs.

By following these steps, you can control how Flask-Appbuilder generates and handles URLs with composite keys. While Flask-Appbuilder doesn’t directly support composite keys out of the box, this approach allows you to manage them effectively.

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