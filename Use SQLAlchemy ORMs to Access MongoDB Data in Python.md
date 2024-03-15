# Use SQLAlchemy ORMs to Access MongoDB Data in Python
_The CData Python Connector for MongoDB enables you to create Python applications and scripts that use SQLAlchemy Object-Relational Mappings of MongoDB data._

The rich ecosystem of Python modules lets you get to work quickly and integrate your systems effectively. With the CData Python Connector for MongoDB and the SQLAlchemy toolkit, you can build MongoDB-connected Python applications and scripts. This article shows how to use SQLAlchemy to connect to MongoDB data to query, update, delete, and insert MongoDB data.

With built-in optimized data processing, the CData Python Connector offers unmatched performance for interacting with live MongoDB data in Python. When you issue complex SQL queries from MongoDB, the CData Connector pushes supported SQL operations, like filters and aggregations, directly to MongoDB and utilizes the embedded SQL engine to process unsupported operations client-side (often SQL functions and JOIN operations).

Connecting to MongoDB Data
--------------------------

Connecting to MongoDB data looks just like connecting to any relational data source. Create a connection string using the required connection properties. For this article, you will pass the connection string as a parameter to the create_engine function.

Set the Server, Database, User, and Password connection properties to connect to MongoDB. To access MongoDB collections as tables you can use automatic schema discovery or write your own schema definitions. Schemas are defined in .rsd files, which have a simple format. You can also execute free-form queries that are not tied to the schema.

Follow the procedure below to install SQLAlchemy and start accessing MongoDB through Python objects.

Install Required Modules
------------------------

Use the pip utility to install the SQLAlchemy toolkit and SQLAlchemy ORM package:

`` `pip install sqlalchemy` ``

`` `pip install sqlalchemy.orm` ``

Be sure to import the appropriate modules:

`` `from` `sqlalchemy` `import` `create_engine, String, Column` ``

`` `from` `sqlalchemy.ext.declarative` `import` `declarative_base` ``

`` `from` `sqlalchemy.orm` `import` `sessionmaker` ``

Model MongoDB Data in Python
----------------------------

You can now connect with a connection string. Use the create_engine function to create an Engine for working with MongoDB data.

_NOTE_: Users should URL encode the any connection string properties that include special characters. For more information, refer to the [SQL Alchemy documentation](https://docs.sqlalchemy.org/en/14/core/engines.html#supported-databases).

``` `engine` `=` `create_engine(``"mongodb:///?Server=MyServer&Port=27017&Database=test&User=test&Password=Password"``)` ```

### Declare a Mapping Class for MongoDB Data

After establishing the connection, declare a mapping class for the table you wish to model in the ORM (in this article, we will model the restaurants table). Use the sqlalchemy.ext.declarative.declarative_base function and create a new class with some or all of the fields (columns) defined.

`` `base` `=` `declarative_base()` ``

`` `class` `restaurants(base):` ``

`` `__tablename__` `=` `"restaurants"` ``

``` `borough` `=` `Column(String,primary_key``=``True``)` ```

`` `cuisine` `=` `Column(String)` ``

`` `...` ``

Download a free trial of the MongoDB Connector to get started:

[ Download Now](https://www.cdata.com/drivers/mongodb/download/python/)

### Query MongoDB Data

With the mapping class prepared, you can use a session object to query the data source. After binding the Engine to the session, provide the mapping class to the session query method.

#### Using the query Method

``` `engine` `=` `create_engine(``"mongodb:///?Server=MyServer&Port=27017&Database=test&User=test&Password=Password"``)` ```

``` `factory` `=` `sessionmaker(bind``=``engine)` ```

`` `session` `=` `factory()` ``

``` `for` `instance` `in` `session.query(restaurants).filter_by(Name``=``"Morris Park Bake Shop"``):` ```

``` `print``(``"borough: "``, instance.borough)` ```

``` `print``(``"cuisine: "``, instance.cuisine)` ```

``` `print``(``"---------"``)` ```

Alternatively, you can use the execute method with the appropriate table object. The code below works with an active session.

#### Using the execute Method

``` `restaurants_table` `=` `restaurants.metadata.tables[``"restaurants"``]` ```

``` `for` `instance` `in` `session.execute(restaurants_table.select().where(restaurants_table.c.Name` `=``=` `"Morris Park Bake Shop"``)):` ```

``` `print``(``"borough: "``, instance.borough)` ```

``` `print``(``"cuisine: "``, instance.cuisine)` ```

``` `print``(``"---------"``)` ```

For examples of more complex querying, including JOINs, aggregations, limits, and more, refer to the Help documentation for the extension.

### Insert MongoDB Data

To insert MongoDB data, define an instance of the mapped class and add it to the active session. Call the commit function on the session to push all added instances to MongoDB.

``` `new_rec` `=` `restaurants(borough``=``"placeholder"``, Name``=``"Morris Park Bake Shop"``)` ```

`` `session.add(new_rec)` ``

`` `session.commit()` ``

### Update MongoDB Data

To update MongoDB data, fetch the desired record(s) with a filter query. Then, modify the values of the fields and call the commit function on the session to push the modified record to MongoDB.

``` `updated_rec` `=` `session.query(restaurants).filter_by(SOME_ID_COLUMN``=``"SOME_ID_VALUE"``).first()` ```

`` `updated_rec.Name` `=` `"Morris Park Bake Shop"` ``

`` `session.commit()` ``

### Delete MongoDB Data

To delete MongoDB data, fetch the desired record(s) with a filter query. Then delete the record with the active session and call the commit function on the session to perform the delete operation on the provided records (rows).

``` `deleted_rec` `=` `session.query(restaurants).filter_by(SOME_ID_COLUMN``=``"SOME_ID_VALUE"``).first()` ```

`` `session.delete(deleted_rec)` ``

`` `session.commit()` ``

Free Trial & More Information
-----------------------------

Download a [free, 30-day trial](https://www.cdata.com/drivers/mongodb/download/python/) of the MongoDB Python Connector to start building Python apps and scripts with connectivity to MongoDB data. Reach out to our [Support Team](https://www.cdata.com/support/submit.aspx) if you have any questions.

Download a free trial of the MongoDB Connector to get started:

[ Download Now](https://www.cdata.com/drivers/mongodb/download/python/)