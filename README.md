# Security Vulnerabilities Project

I was given an existing codebase containing a number of security vulnerabilities

## The goal

I had to discover and rectify all security vulnerabilities utilising SAST (Bandit) and DAST (OWASP ZAP) tools.

## Security vulnerabilities found

user.py------------------------------------------------------------------------------------------------------------------------------------

- Lines 10, 18, 29 - User inputs needed to be escaped and so placeholders can be used. This is used to counteract the risk of SQL injection.

```python
"INSERT INTO user (username, password) VALUES ('"+username+"', '"+password+"')"
```
```python
"INSERT INTO user (username, password) VALUES (?, ?)", [username, password]
```

__init__.py--------------------------------------------------------------------------------------------------------------------------------

- Line 8 - Secret key should not be hard coded into the codebase. Instead it should be defined as an environment variable. This would require
    you to import the 'os' module. You would then need to store it as an environment variable before running the application.

```python
SECRET_KEY='super_secret_key'
```
```python
SECRET_KEY=os.environ.get('SECRET_KEY')
```

schema.sql, user.py------------------------------------------------------------------------------------------------------------------------

- Line 5 - When a user is added to the database, the ID is autoincremented. This would make it easier to guess the ID of other users. In this
    case you could make use of the 'uuid' module to create a random Universaly Unique Indentifier. You would then assign this number as the ID
    in your database. You would need to change the type in the DB schema as well as the method in the user class in user.py. Although there is
    no test to check that it is unique, the number of different variations of UUID are astronomical so it is perhaps not necessary.

```SQL
id INTEGER PRIMARY KEY AUTOINCREMENT
```
```SQL
id TEXT PRIMARY KEY
```

```python
  def create(cls, username, password):
    db = get_db()
    db.execute(
      "INSERT INTO user (username, password) VALUES (?, ?)", [username, password])
```
```python
import uuid

def create(username, password):
    user_id = str(uuid.uuid4())
    db = get_db()
    db.execute(
      "INSERT INTO user (id, username, password) VALUES (?, ?, ?)", [user_id, username, password])
```

login.html, register.html------------------------------------------------------------------------------------------------------------------

- Line 18 - As it stands, the way these HTML files are written means that the site is at risk of Cross-Site Request Forgery (CSRF) attacks. To fix 
  this   vulnerability, you need to implement anti-CSRF tokens in your forms. These tokens are used to verify that the requests submitted to your 
  server are legitimate and originated from your own site, helping to prevent CSRF attacks. This would be done by importing CSRFProtect from the Flask WTF 
  module which would generate a unique session token for each user session. This would then be passed into the HTML file using a hidden input field.

```python
def create_app(test_config=None):
    # create and configure the app
    app = Flask(__name__, instance_relative_config=True)
```

```python
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect()

def create_app(test_config=None):
    # create and configure the app
    app = Flask(__name__, instance_relative_config=True)
    csrf.init_app(app)
```

```HTML
<section class="content">
    <form method="post">
        <label for="username">Username</label>
```

```HTML
<section class="content">
    <form method="post">
      <input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>
      <label for="username">Username</label>
```

__init__.py--------------------------------------------------------------------------------------------------------------------------------

- Line 45 - The Content Security Policy (CSP) is a security standard that helps prevent various types of attacks, such as Cross-Site Scripting (XSS) 
  and data injection attacks. CSP defines a set of rules that control the resources a web page is allowed to load, such as scripts, stylesheets, 
  images, and more. As things stand, the CSP defined in our application is too broad and as such is potentially susceptible to attack. In order to
  rectify this, we need to tailor the CSP to make it more secure. This also helps protect against ClickJacking attacks.

```python
@app.after_request
    def add_security_headers(resp):
        resp.headers['Content-Security-Policy']='default-src \'self\''
        return resp
```
```python
    @app.after_request
    def add_security_headers(resp):
        resp.headers['Content-Security-Policy'] = 'default-src \'self\'; script-src \'self\'; style-src \'self\'; frame-ancestors \'self\'; form-action \'self\''
        return resp
```

__init__.py--------------------------------------------------------------------------------------------------------------------------------

- Line 10-16 - In the #create_app function, there is currently no SameSite cookie configuration. SameSite is an attribute that can be set on cookies to
  control their behaviour with regard to cross-site requests. It is a security feature that helps mitigate certain types of attacks, including Cross-Site 
  Request Forgery (CSRF) and Cross-Site Scripting (XSS). In order to fix this, you need to add another line to the app configuration.

```python
    app.config.from_mapping(
        SECRET_KEY=os.environ.get('SECRET_KEY'),
        DATABASE=os.path.join(app.instance_path, 'login_form.sqlite'))
```
```python
    app.config.from_mapping(
        SECRET_KEY=os.environ.get('SECRET_KEY'),
        DATABASE=os.path.join(app.instance_path, 'login_form.sqlite'),
        SESSION_COOKIE_SAMESITE='Lax')
```

__init__.py--------------------------------------------------------------------------------------------------------------------------------

- Line 44-47 - There is currently no protection again MIME type sniffing attacks. MIME type sniffing is a security risk where browsers may interpret a file
as a different MIME type than what the server specified. In order to fix this vulnerability, you need to add another security header. This will tell the browser
to not perform MIME type sniffing and to respect the declared content type.

```python
    @app.after_request
    def add_security_headers(resp):
        resp.headers['Content-Security-Policy'] = 'default-src \'self\'; script-src \'self\'; style-src \'self\'; frame-ancestors \'self\'; form-action \'self\''
```

```python
    @app.after_request
    def add_security_headers(resp):
        resp.headers['Content-Security-Policy'] = 'default-src \'self\'; script-src \'self\'; style-src \'self\'; frame-ancestors \'self\'; form-action \'self\''
        resp.headers['X-Content-Type-Options'] = 'nosniff'
```
