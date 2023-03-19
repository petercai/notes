# Why FastAPI is a Future of Python Web Development
Exploring the Powerful Features and Advantages of FastAPI Framework for Building Python-Based Web Applications
--------------------------------------------------------------------------------------------------------------

![](https://miro.medium.com/v2/resize:fit:700/1*du7p50wS_fIsaC_lR18qsg.png)

**Introduction**

If you are a Python Back-end developer or at least somehow connected to the field you must have heard about FastAPI. It’s a clear trend in the industry. In this article, we will try to identify the main reasons for that and maybe it will become your framework of choice.

**What is FastAPI anyway?**

FastAPI is a modern, fast (hence the name) web framework for building APIs with Python. It is built on top of the Starlette framework and relies heavily on Python’s type annotations to provide fast and efficient validation, serialization, and documentation of API requests and responses.

**What are the main features of FastAPI?**

*   Fast performance: FastAPI is built with high-performance libraries such as Pydantic and Uvicorn to achieve incredible speeds and handle high traffic loads.
*   Simple and easy-to-use: FastAPI comes with a simple and intuitive API that is easy to learn and use.
*   Easy validation and serialization: FastAPI uses Python’s type hints to automatically validate API requests and responses, as well as to automatically serialize and deserialize data to and from JSON, eliminating boilerplate code.
*   Built-in documentation: FastAPI automatically generates interactive API documentation, allowing developers to quickly and easily explore and test APIs.
*   ASGI support: FastAPI is built on top of ASGI (Asynchronous Server Gateway Interface), allowing it to leverage the power of asynchronous programming for even faster performance.
*   Dependency injection: FastAPI makes use of Python’s dependency injection system to automatically inject dependencies into API endpoints and other components, making it easy to build complex applications.

Let’s get into more detail about some of them.

**Easy validation and serialization**

In my opinion, this is the greatest part of FastAPI. The key here is that FastAPI uses _built-in Python syntax_ to achieve these things. Here’s an example:

```
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")  
async def read_item(item_id: int):  
    return {"item_id": item_id}


```

That’s a pretty simple API endpoint that takes _item_id_ as an input and returns it in JSON format. The interesting thing here is that Python’s hints are used to validate the input. So if you would call this endpoint like that:

> “/items/test”

It would return a validation error:

```
{  
    "detail": \[  
        {  
            "loc": \[  
                "path",  
                "item_id"  
            \],  
            "msg": "value is not a valid integer",  
            "type": "type_error.integer"  
        }  
    \]  
}
```

_But what if you need something more complex than that?_

**Pydantic**

> Data validation and settings management using Python type annotations. _pydantic_ enforces type hints at runtime, and provides user friendly errors when data is invalid. Define how data should be in pure, canonical Python; validate it with _pydantic_.

Here’s the example of using Pydantic in FastAPI application:

```
from typing import Union

from fastapi import FastAPI  
from pydantic import BaseModel

class Item(BaseModel):  
    name: str  
    description: Union\[str, None\] = None  
    price: float  
    tax: Union\[float, None\] = None

app = FastAPI()

@app.post("/items/")  
async def create_item(item: Item):  
    return item


```

Still works with built-in type hints. As well as Pydantic model itself uses type hints to describe the fields. If you only worked with frameworks like Django and Flask this should already amaze you.

Let’s see the example of using Django REST Framework’s serializers to achieve the same result:

```
class CommentSerializer(serializers.Serializer):  
    name = serializers.CharField(max_length=200)  
    description = serializers.CharField(max_length=500, required=False)  
    price = serializers.FloatField()  
    tax = serializer.FloatField(required=False)
```

This is a simple example of DRF’s serializer and of course, it’s working, but Pydantic’s version is a cleaner and more **_pythonic_** way.

Serialization is even simpler, we can just return an instance of Pydantic’s model from the _path operation function_ and it will be automatically serialized with all the nested models.

**Automatic docs**

FastAPI will generate OpenAPI schema based on your type hints, Pydantic models, _and path operation function’_s parameters and it will be shown in the interactive API docs.

![](https://miro.medium.com/v2/resize:fit:700/1*4aVgUyw3J7tREZhAArXRLw.png)

FastAPI gives massive opportunities to modify and configure documentation based on your needs. You can add descriptions, comments, titles, examples, and everything you can think of with clean syntax.

**Editor support**

This feature is a repercussion of the points listed above.

![](https://miro.medium.com/v2/resize:fit:700/1*KYTSq-y4eHMxCYotu-01eg.png)

Editor support may sound pretty useless at first glance, but believe me — that’s a live saver for big projects. When your project is growing, you should keep your data models apart from business logic, as well as endpoints (I hope you do), so it’s pretty cool that editor already knows what properties your model has of what types, so you will avoid a lot of stupid mistakes while coding. People tend to underestimate this point, but it can become a real thing when you push a small bug into production which could be avoided using editor highlighting.

**Dependency injection**

> In [software engineering](https://en.wikipedia.org/wiki/Software_engineering), **dependency injection** is a [design pattern](https://en.wikipedia.org/wiki/Software_design_pattern) in which an [object](https://en.wikipedia.org/wiki/Object_(computer_science)) or [function](https://en.wikipedia.org/wiki/Subroutine) receives other objects or functions that it depends on. A form of [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control), dependency injection aims to [separate the concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) of constructing objects and using them, leading to [loosely](https://en.wikipedia.org/wiki/Loose_coupling) [coupled](https://en.wikipedia.org/wiki/Coupling_(computer_programming)) programs.

Dependency injection is a widely used design pattern in other programming languages, but not in Python. It’s coming from full OOP languages like Java and .NET, so you might not be even familiar with it, but what do you think? It’s also a lifesaver.

Let’s see how it works in FastAPI.

To implement DI we need to have a “dependable”, which in FastAPI is every function (sync, or async) that can take all the same parameters that a _path operation function_ can take.

```
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()

async def common_parameters( q: Union\[str, None\] = None, skip: int = 0, limit: int = 100):  
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")  
async def read_items(commons: dict = Depends(common_parameters)):  
    return commons

@app.get("/users/")  
async def read_users(commons: dict = Depends(common_parameters)):  
    return commons


```

Here common_parameters is a _dependable_ and read\_items, read\_users are _dependants_.

Before executing read\_items’ code, common\_parameters will be executed.

You can use multiple dependencies in one function and dependable can also have a dependency, so basically you can build trees of them to match your needs.

The interesting part for me here is that you don’t need to worry about parameters that your dependable receives at all. FastAPI will automatically take care of passing the right parameters to your functions even if _the parent_ doesn’t have access to it. See this example:

```
  
from typing import Union

from fastapi import Depends, FastAPI, Cookie

app = FastAPI()

async def common_parameters( q: Union\[str, None\] = None, skip: int = 0, limit: int = 100):  
    return {"q": q, "skip": skip, "limit": limit}

def check_cookies(ads_id: Union\[str, None\] = Cookie(default=None)):  
    return {"ads_id": ads_id}

@app.get("/items/")  
async def read_items( commons: dict = Depends(common_parameters), cookie: dict = Depends(check_cookies)):  
    return commons


```

read_items function does not know anything about _cookies_ themselves because we didn’t specify that parameter, but the dependable function is still able to get access to that _cookie_. This is also a very strong point about FastAPI because it helps to keep the code so much cleaner and professional and still follows the rule about using python build-in syntax as much as possible.

**Clean code**

All points above are great on their own, but the most important part is that they are leading (if not forcing) to **_clean code habits_**. I’ve seen python developers that don’t even know what are _type annotations_ and why we need them, I’ve seen a lot of applications that have a lot of _duplicated code_. FastAPI makes it mandatory to write type hints, declare dependencies, and follow OOP design patterns which help to create _great code structure_ and _keep code clean_. This is the reason why it’s so popular now and I feel it’s here to stay.

**Conclusion**

In conclusion, FastAPI is a powerful and efficient Python web framework that has been gaining popularity in recent years. With its impressive speed, simple API, and built-in documentation, FastAPI is an excellent choice for building high-performance APIs. Its use of Python’s type hints for automatic validation and serialization eliminates boilerplate code and makes development faster and easier. Additionally, FastAPI’s support for ASGI and dependency injection further enhances its capabilities, making it a top choice for both beginner and experienced developers. Overall, if you’re looking for a modern and reliable Python web framework, FastAPI is definitely worth considering.