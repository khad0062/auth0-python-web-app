# CST8919 Lab 1: Implementing User Login with Flask and Auth0

## Overview
This Flask application demonstrates secure authentication using Auth0. Users can log in/log out and access protected routes. The application acts as a Service Provider (SP) while Auth0 serves as the Identity Provider (IdP).

## Features
- Auth0 authentication integration
- Login/logout functionality
- Protected route (`/protected`) for authenticated users only
- Session management

## Setup Instructions

### 1. Prerequisites
- Python 3.9+
- [Auth0 account](https://auth0.com/signup)

### 2. Configure Auth0
1. Create a new application in [Auth0 Dashboard](https://manage.auth0.com/)
2. Set application type: "Regular Web Application"
3. Configure settings:
   - **Allowed Callback URLs**: `http://localhost:3000/callback`
   - **Allowed Logout URLs**: `http://localhost:3000`
   - **Allowed Web Origins**: `http://localhost:3000/`
4. Note your credentials:
   - Domain 
   - Client ID
   - Client Secret
Copy them to .env file

### 3. Project Setup

#### Clone repository
```
git clone https://github.com/khad0062/auth0-python-web-app.git
```
cd autho-python-web-app

#### Create virtual environment (Windows)
```
python -m venv venv
venv\Scripts\activate
```

#### Install dependencies
```
pip install -r requirements.txt
```

#### 4. Environment Variables
Create .env file in project root:
```
AUTH0_DOMAIN=your-domain.us.auth0.com
AUTH0_CLIENT_ID=your_client_id
AUTH0_CLIENT_SECRET=your_client_secret
APP_SECRET_KEY=your_random_secret_key
```
- Generate a suitable string for APP_SECRET_KEY using openssl rand -hex 32 from your shell.
#### Create a server.py
```python
import json
from os import environ as env
from urllib.parse import quote_plus, urlencode

from authlib.integrations.flask_client import OAuth
from dotenv import find_dotenv, load_dotenv
from flask import Flask, redirect, render_template, session, url_for
from functools import wraps

# Load .env
ENV_FILE = find_dotenv()
if ENV_FILE:
    load_dotenv(ENV_FILE)

# App setup
app = Flask(__name__)
app.secret_key = env.get("APP_SECRET_KEY")

# Auth0 setup
oauth = OAuth(app)
oauth.register(
    "auth0",
    client_id=env.get("AUTH0_CLIENT_ID"),
    client_secret=env.get("AUTH0_CLIENT_SECRET"),
    client_kwargs={"scope": "openid profile email"},
    server_metadata_url=f'https://{env.get("AUTH0_DOMAIN")}/.well-known/openid-configuration',
)

# Auth required decorator
def requires_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if "user" not in session:
            return redirect(url_for("login"))
        return f(*args, **kwargs)
    return decorated

# Routes
@app.route("/")
def home():
    return render_template(
        "home.html",
        session=session.get("user"),
        pretty=json.dumps(session.get("user"), indent=4),
    )

@app.route("/login")
def login():
    return oauth.auth0.authorize_redirect(
        redirect_uri=url_for("callback", _external=True)
    )

@app.route("/callback", methods=["GET", "POST"])
def callback():
    token = oauth.auth0.authorize_access_token()
    session["user"] = token
    return redirect("/")

@app.route("/logout")
def logout():
    session.clear()
    return redirect(
        "https://"
        + env.get("AUTH0_DOMAIN")
        + "/v2/logout?"
        + urlencode(
            {
                "returnTo": url_for("home", _external=True),
                "client_id": env.get("AUTH0_CLIENT_ID"),
            },
            quote_via=quote_plus,
        )
    )

@app.route("/protected")
@requires_auth
def protected():
    return render_template(
        "protected.html",
        session=session.get("user"),
        pretty=json.dumps(session.get("user"), indent=4),
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=env.get("PORT", 3000))
```

##### Protected Route
- Access /protected after login to view restricted content.
- Unauthenticated users are automatically redirected to login.
- Implemented with route decorator:

```python
def requires_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if 'profile' not in session:
            return redirect('/login')
        return f(*args, **kwargs)
    return decorated

@app.route('/protected')
@requires_auth
def protected():
    return render_template('protected.html')
```
#### 5. Run the Application
```
python3 server.py
``` 
Visit http://localhost:5000 in your browser
