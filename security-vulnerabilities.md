user.py--------------------------------------------------------------------------------------------------------------------------------------

- Lines 10, 18, 29 - User inputs needed to be escaped and so placeholders can be used. This is used to counteract the risk of SQL injection.

```python
"INSERT INTO user (username, password) VALUES ('"+username+"', '"+password+"')"
```
```python
"INSERT INTO user (username, password) VALUES (?, ?)", [username, password]
```

__init__.py-----------------------------------------------------------------------------------------------------------------------------------

- Line 8 - Secret key should not be hard coded into the codebase. Instead it should be defined as an environment variable. This would require
    you to import the 'os' module. You would then need to store it as an environment variable before running the application.

```python
SECRET_KEY='super_secret_key'
```
```python
SECRET_KEY=os.environ.get('SECRET_KEY', 'super_secret_key')
```