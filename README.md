# Google OAuth2 Authentication Guide

This documentation outlines the step-by-step implementation of Google OAuth2 authentication for your app using Symfony and Docker.

---

Visit this site to get your GOOGLE_CLIENT_ID and GOOGLE_CLIENT_SECRET

```bash
https://console.cloud.google.com/auth/clients
```

## 1. Installation

Install the required bundles via Composer to handle OAuth2 clients and email verification.

```bash
composer require knpuniversity/oauth2-client-bundle
composer require league/oauth2-google
```
optional if you accidentally click yes email verifier

```bash
composer require symfonycasts/verify-email-bundle
```

---

## 2. Google Cloud Console Setup

Configure your credentials at Google Cloud Console.

- **Authorized Redirect URI:**
```
http://127.0.0.1:8000/connect/google/check
```

- **CORS Policy (.env):**

```bash
#put this on your .env
CORS_ALLOW_ORIGIN='^https?://(localhost|127\.0\.0\.1)(:[0-9]+)?$'
```

---

## 3. Environment Configuration

### Database (Docker)

Optional namin convention since it is intended for oauth2

```bash
# .env configuration
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=Oauth
MYSQL_USER=googleuser
MYSQL_PASSWORD=dbpassword
MYSQL_HOST_PORT=3310
MYSQL_CONTAINER_PORT=3306
MYSQL_PORT=${MYSQL_HOST_PORT}:${MYSQL_CONTAINER_PORT}

PMA_HOST=mysql
PMA_USER=googleuser
PMA_PASSWORD=googlepass
```

### Google API Keys

Secure this credentials on Google Cloud Console

`Paste it on your .env or .env.local`

```bash
GOOGLE_CLIENT_ID=your_client_id_here
GOOGLE_CLIENT_SECRET=your_client_secret_here
```

---

## 4. System Configuration

### Client Registration

Configure the OAuth2 client in `config/packages/knpu_oauth2_client.yaml`:

```yaml
knpu_oauth2_client:
  clients:
    google:
      type: google
      client_id: '%env(GOOGLE_CLIENT_ID)%'
      client_secret: '%env(GOOGLE_CLIENT_SECRET)%'
      # This route must generate the URI you put in Google Console
      redirect_route: connect_google_check
      redirect_params: {}
```

---

### Security Layer

Update `config/packages/security.yaml`:

```yaml
security:
    firewalls:
        main:
            custom_authenticators:
                - App\Security\LoginFormAuthenticator
                - App\Security\GoogleAuthenticator
            user_checker: App\Security\UserChecker
```

---

## 5. Implementation: User Verification

The `UserChecker.php` handles pre-authentication logic, ensuring only active and verified users can log in.

| Condition           | Action                                         |
|--------------------|-----------------------------------------------|
| Account Inactive   | Throws Deactivated exception                  |
| Email Not Verified | Throws exception unless the provider is Google|

---

## 6. Frontend Integration

Place the following button in your Twig login template
I'm using Tailwind here:

```twig
<div class="auth-wrapper">
    <a href="{{ path('connect_google_start') }}" id="google-login-link" class="btn-google">
        <ion-icon name="logo-google" class="icon-left"></ion-icon>
        Continue with Google
    </a>
</div>
```
If you were using css:

```twig
<div class="auth-wrapper">
    <a href="{{ path('connect_google_start') }}" id="google-login-link" class="btn-google">
        <span class="icon-left">
            <ion-icon name="logo-google"></ion-icon>
        </span>
        <span>Continue with Google</span>
    </a>
</div>
```

```css
.auth-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 20px;
}

.btn-google {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    padding: 10px 16px;
    background-color: #ffffff;
    color: #444;
    text-decoration: none;
    font-size: 14px;
    font-weight: 500;
    border: 1px solid #ddd;
    border-radius: 6px;
    box-shadow: 0 1px 2px rgba(0,0,0,0.1);
    transition: all 0.2s ease;
}

.btn-google:hover {
    background-color: #f7f7f7;
    border-color: #ccc;
}

.icon-left {
    display: flex;
    align-items: center;
    font-size: 18px;
}
```

Optional icon libraries
```js
<script type="module" src="https://unpkg.com/ionicons@7/dist/ionicons/ionicons.esm.js"></script>
<script nomodule src="https://unpkg.com/ionicons@7/dist/ionicons/ionicons.js"></script>
```
---

## Troubleshooting & Utilities

### Manual Database Permission Fix

If the Docker container denies access to `googleuser`, run these commands inside the container:

```bash
#If ever you encounter REQUIRED PASSWORD 'YES' then heres the fix open CMD

docker ps #this command help you determine 'your_container_name'.

# 1. Enter the container
docker exec -it <your_container_name> mysql -u root -p

# 2. Grant privileges (SQL)
CREATE USER IF NOT EXISTS 'googleuser'@'%' IDENTIFIED BY 'dbpassword';
GRANT ALL PRIVILEGES ON Oauth.* TO 'googleuser'@'%';
FLUSH PRIVILEGES;
```

---

### Git Maintenance

To undo a commit while keeping your code changes staged:

```bash
git reset --soft HEAD~1
```