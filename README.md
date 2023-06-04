![image](https://github.com/kohutd/kuuAuth/assets/21020331/e4c8f0e3-6e0f-4945-bc4a-1cf0423d454c)

# kuuAuth

Know Your User Authentication - is an approach to user authentication that allows you to easily track user's activity before he got logged or registered.

## How it works

Basically you check or create a session upon application start and check the session upon other actions.

If user was logged in before then session is authenticated. If not then the session is guest-session.

After user successfully logs in using guest session, the session becomes authenticated and session token is refreshed.

## Flow

### Guest

#### first touch

In this step user was never on our application and a server creates new guest session for him.

Request:

```http
POST /start HTTP/2
```

Response:

```http
HTTP/2 200 OK
X-MyAwesomeApplication-Session-Token: some_random_bytes
Content-Type: application/json

{"@type":"GuestSessionResult"}
```


#### second touch

In this step user opens our application second time and the server has no need to create new guest session.

Request:

```http
POST /start HTTP/2
X-MyAwesomeApplication-Session-Token: some_random_bytes
```

Response:

```http
HTTP/2 200 OK
Content-Type: application/json

{"@type":"GuestSessionResult"}
```

#### login/register

In this step user tries to login or register within guest session. Guest session becomes authenticated and it's token gets refreshed.

Request:

```http
POST /login?email=david@storinka&password=password HTTP/2
X-MyAwesomeApplication-Session-Token: some_random_bytes
```

Response:

```http
HTTP/2 200 OK
X-MyAwesomeApplication-Session-Token: some_new_random_bytes
Content-Type: application/json

{"@type":"UserSessionResult"}
```

**If user tries to login or register without session then the server should give an error.**

Request:

```http
POST /login?email=david@storinka&password=password HTTP/2
```

Response:

```http
HTTP/2 400 Bad Request
Content-Type: application/json

{"error":"SESSION_REQUIRED"}
```

**If user tries to login or register within authenticated session then the server should give an error.**

Request:

```http
POST /login?email=david@storinka&password=password HTTP/2
X-MyAwesomeApplication-Session-Token: some_new_random_bytes
```

Response:

```http
HTTP/2 400 Bad Request
Content-Type: application/json

{"error":"SESSION_ALREADY_AUTHENTICATED"}
```

**If user tries to call any method but `/start` without a session then the server should give an error.**

Request:

```http
POST /getMusic HTTP/2
```

Response:

```http
HTTP/2 400 Bad Request
Content-Type: application/json

{"error":"SESSION_REQUIRED"}
