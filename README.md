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
   - **Allowed Callback URLs**: `http://localhost:5000/callback`
   - **Allowed Logout URLs**: `http://localhost:5000`
   - **Allowed Web Origins**: `http://localhost:5000`
4. Note your credentials:
   - Domain 
   - Client ID
   - Client Secret
     Copy them to .env file

### 3. Project Setup

# Clone repository
```git clone https://github.com/khad0062/auth0-python-web-app.git
```
cd autho-python-web-app

# Create virtual environment (Windows)
python -m venv venv
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

### 4. Environment Variables
Create .env file in project root:
```
AUTH0_DOMAIN=your-domain.us.auth0.com
AUTH0_CLIENT_ID=your_client_id
AUTH0_CLIENT_SECRET=your_client_secret
APP_SECRET_KEY=your_random_secret_key
```

### 5. Run the Application
```
python3 server.py
``` 
Visit http://localhost:5000 in your browser


## Protected Route
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
