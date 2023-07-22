1. Flask CLI
	1. export FLASK_APP=main.py
	2. flask run
	3. flask shell
	   
2. Flask app configuration
For now, we will create a config file
first. In the file named config.py, add the following:

```
class Config(object):
  pass
class ProdConfig(Config):
  pass
class DevConfig(Config):
  DEBUG = True
```
Now, in another file named main.py, add the following:

```
from flask import Flask
from config import DevConfig

app = Flask(__name__)
app.config.from_object(DevConfig)

@app.route('/')
def home():
  return 'Hello World!'
if __name__ == '__main__':
  app.run()
```

2. database - creatting models with SQLAlchemy
	1. ORM: SQLAlchemy (Flask-Alchemy)
	2. Migration: Alembic (Flask-Migrate)
	3. Alembic CLI
		1. flask db
		2. flask db init
		3. flask db migrate -m"initial migration"
			1. This command will cause Alembic to scan our SQLAlchemy object and find all the tables and columns that did not exist before this commit. As this is our first commit, the migration file will be rather long. Be sure to specify the migration message with -m, as it's the easiest way to identify what each migration is doing. Each migration file is stored in the migrations/versions/ folder.
		 4. flask db upgrade
		 5. flask db upgrade --sql
		 6. flask db history
		 7. flask db downgrade <uuid>
		4. database management with Flask-Admin
 1. creating views with templates - Jinja
    - In Jinja,variable substitutions are defined by {{ }}. The {{ }} syntax is called a variable block.
    - There are also control blocks defined by {% %} that declare language functions, such asloops or if statements.
    - WTForms is a library that handles sever form validation for you by checking input against common form types. Flask WTForms is a Flask extension that is built on top of WTForms that adds features, such as Jinja HTML rendering, and protects you against attacks, such as SQL injection and cross-site request forgery
      
4. creating controllers with Blueprints
	1. **Sessions** are the way Flask will store information across requests; to do this, Flask will usesigned cookies using the previously set SECRET_KEY config to apply the HMAC-SHA1 default cryptographic method. So, a user can read their session cookie but can't modify it. Flask also sets a default session lifetime that defaults to 31 days to prevent relay attacks; this can be changed by using the configuration key's PERMANENT_SESSION_LIFETIME config key.
	2. **Global** is a thread-safe namespace store to keep data during a request's context. At the beginning of each request, a new global object is created, and at the end of the request the object is destroyed.