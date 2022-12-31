# JSON-Based Serialized LOB Pattern - DZone
_Serialized LOB_ (Large Object) pattern is explained in the [_Patterns of Enterprise Application Architecture_](https://martinfowler.com/books/eaa.html) book by Martin Fowler. The original paper describes an implementation of the pattern with an example of XML-based serialization. Although nowadays, JSON format is ubiquitous. So, adding a fresh flavor of “JSONization” to the pattern seems fully justifiable.

There was mentioned that a binary serialization might win in size and performance, though the main problem with BLOBs ([Binary Large Objects](https://dzone.com/articles/performance-testing-blob-from-a-mysql-database-wit)) remains the versioning. The changes in classes’ structure may lead to data incompatibility, so it can be problematic to de-serialize your objects from the database. Also, it is hardly achievable to query data inside BLOB. On the other hand, CLOB (Character Large Object) type is human readable, which at least allows for investigating issues. In addition, modern [RDBM](https://dzone.com/articles/a-look-at-the-history-of-rdbms) systems provide instruments to manipulate [JSON](https://dzone.com/articles/introduction-to-json-with-java), like querying and modifying data inside JSON.

Before diving into implementation details, let’s review the objectives of the pattern:

*   Simplifying a database structure by decreasing the number of related tables. This helps to reduce the number of joins and works better with small and immutable data. In general, this might be considered database denormalization.
*   Grouping table columns that together represent some object, and this object is an entity’s attribute. In other words, nonprimitive types are serialized and stored in a single column instead of multiple ones.
*   Arrange an entity’s attributes by their importance or by the frequency of usage. For example, key attributes that identify entities and participate in search should be mapped onto their own columns – a column per attribute. Other attributes, which are used occasionally, might be combined into a helper object, which is serialized and presented in the database as a single column.

Query Simplification
--------------------

Imagine a site that shows basic information about products. At the minimum, `products` the table would have the structure:![](https://dz2cdn1.dzone.com/storage/temp/16604551-serialized-lob-implementation-productsdrawio.png)

If the site supports multiple languages, a localization might end up with something like this nightmare:

![](https://dz2cdn1.dzone.com/storage/temp/16604594-serialized-lob-implementation-products-localizatio.png)

Or with the better solution that uses a separate localization table:

![](https://dz2cdn1.dzone.com/storage/temp/16604554-serialized-lob-implementation-multiple-languagesdr.png)

The first variant is difficult to extend, and retrieving data by a locale would be awkward enough to avoid this option. [SQL](https://dzone.com/articles/understanding-sql-dialects) request for the second variant may be like this:

We can select data for a required locale, though we must join three tables.

With _Serialized LOB_ pattern, it is possible to use an original structure of `products` the table. What is changed is the meaning and content of text columns. Instead of storing a single value of an attribute for some locale, the columns contain JSON string that presents `LocalizedString` an object, which in Java class with [Lombok](https://projectlombok.org/) annotations and convenient method `getByLocale` might be implemented as the following:

An example of JSON presentation of `name` attribute would like `{“en”:”Hand plow”,”it”:”Aratro a mano”,”de”:”Handpflug”}`.

Both `name` and `description` JSON can be stored directly in corresponding columns. So, it is possible to select data just by-product `id` without extra joins. Though this requires serialization/deserialization to and from JSON strings. A bit later, we will see how it can be implemented.

Object-Oriented Strategy
------------------------

_Serialized LOB_ pattern may also be helpful when your entity classes have attributes of object types. For instance, if you need to add size columns into `products` table, this can be done like this:

![](https://dz2cdn1.dzone.com/storage/temp/16604556-serialized-lob-implementation-products-sizedrawio.png)

But logically, these four new attributes present dimensions of the product. So, with the [OO](https://dzone.com/articles/object-oriented-programming-concepts-with-a-system) (object-oriented) approach, the following looks less verbose:

![](https://dz2cdn1.dzone.com/storage/temp/16604557-serialized-lob-implementation-products-dimensiondr.png)

Where the column dimensions store the JSON presentation of `Dimensions` object:` {"unit":{"en":"meter","fr":"mètre","it":"metro"},"length":"1","width":"2","height":"3"}`.

The OO approach works perfectly with immutable objects of stable, non-modifiable classes. Another example of such an immutable class is `Location` which might be a part of `Address` class:

JSON presentation of `Location` the object looks like this: `{"latitude":"-15.95615673677379", "longitude":"-5.69888813195389"}`.

Multiple Attribute Packing
--------------------------

This approach is similar to the previous one – the packing attributes of an entity into an object, which is serialized into JSON. However, the requirements of the immutability of serialized classes might be dropped. The main reason to use patterns here is convenience.

For instance, you have `Store` a class with dozens of attributes. Some of them are necessary for identifying, searching, or displaying, like `code`, `name`, `description`. Others might be used only in rare specific cases, for example, `defaultStorage`, `isEligibleForHomeDelivery`, `timeZone`, etc. As a result, the `stores` the table might have so many columns that make it inconvenient to operate with. Instead of working with multiple queries, it is suggested to split attributes into groups. Key attributes are mapped onto dedicated columns, whereas other attributes, which are not used often, will be packed into a helper class as shown below:

![](https://dz2cdn1.dzone.com/storage/temp/16604558-serialized-lob-implementation-storesdrawio.png)

Where the column `store_details` contains the JSON presentation of an object of `StoreDetails` class:

Pay attention to the last two attributes. It is quite safe to use fields of quasi-final types, like `Location`, whereas the usage of mutable classes, like `Address`, is rather risky. Firstly, _Serialized LOB_s might contain copies of the same data, e.g., the same address shared by different stores. Secondly, if a serialized LOB contains mutable objects inside JSON, the changes in those object classes may break compatibility between versions. For example, if JSON contains a fully serialized store address, `StoreDetails` the object cannot be restored from the JSON string if some attribute is removed from `Address` class.

JSON Serialization
------------------

Now let’s see how the pattern can be implemented with JSON serialization.

An obvious and straightforward technique, especially if you do not use ORM, is to serialize and de-serialize with the help of [Gson](https://github.com/google/gson) or [Jackson](https://github.com/FasterXML/jackson) libraries. With that, while communicating with the DAO layer, you have to convert objects into JSON strings or parse JSON back from the string to the object. This is a simple approach, though it requires an implementation of custom logic in DAO.

A more elegant way to work with JSON data can be implemented if you use JPA and _Hibernate_. Actually, you do not need to create any intermediate layers at all. Instead, with the help of an excellent project [hibernate-types](https://github.com/vladmihalcea/hibernate-types), all you ought to do is declare and annotate your entities correspondingly. All the heavy lifting for converting to/from JSON will be done by custom type definitions which are provided by the hibernate-types project.

Here are the updated definitions of entities mentioned earlier for _Hibernate 5_:

In the fragment above, it is shown how to declare `JsonStringType` the type to use with `Product` entity. Another approach is to place this declaration into a package info file. This is explained in the [hibernate-types](https://github.com/vladmihalcea/hibernate-types) project documentation.

After declaration, the type can be used in annotations of attributes that are to be serialized into JSON when an entity is saved into the database and which are restored from JSON strings into Java objects. The above `LocalizedString` is annotated as `json` type. Highly likely, this class is used by multiple entities which require localized attributes. So, `LocalizedString` is declared as an independent class. On the other hand, `Dimensions` is defined as an embedded class, as hard it will be used in more than one entity.

Finally, here is an extended example of the definition `Store` the entity that uses all types of _Serialized LOB_ patterns:

Again, beware of using modifiable or referenced types inside objects which are the subject to _Serialized LOB_.

Conclusion
----------

Prudently used, _Serialized LOB_ pattern suggests an elegant and solid OO solution. With an implementation based on _Hibernate_ custom types, all JSON manipulations are hidden behind annotations. So, a developer may work with objects not thinking about the mapping between classes and database tables or columns and perhaps even not suspecting that some objects are persisted as strings.

Database Decentralized autonomous organization Desktop environment JSON Attribute (computing) Data (computing) Object (computer science) sql Strings Data Types

Opinions expressed by DZone contributors are their own.