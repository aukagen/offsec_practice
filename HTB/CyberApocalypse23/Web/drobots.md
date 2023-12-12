
Vulnerable SQL code - database.py
```
def login(username, password):
    # We should update our code base and use techniques like parameterization to avoid SQL Injection
    user = query_db(f'SELECT password FROM users WHERE username = "{username}" AND password = "{password}" ', one=True)

    if user:
        token = createJWT(username)
        return token
    else:
        return False

```
An attempt was made to strengthen the code and reduce the possibility of SQLi by using JSON. However, this was not impossible to bypass, thus by escapping the '} character in the json, a union select was done to bypass admin login, which then made the flag available.