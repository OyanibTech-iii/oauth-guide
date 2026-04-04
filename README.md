# Google OAuth2 Authentication Setup (Symfony + Docker)

This documentation outlines the step-by-step implementation of Google OAuth2 authentication for the Growfico platform using Symfony and Docker.

---

## 🛠️ 1. Installation

Install the required bundles via Composer to handle OAuth2 clients and email verification.

```bash
composer require knpuniversity/oauth2-client-bundle
composer require league/oauth2-google
composer require symfonycasts/verify-email-bundle
```

---

## 🔐 2. Google Cloud Console Setup

Configure your credentials at Google Cloud Console.

- **Authorized Redirect URI:**
```
http://127.0.0.1:8000/connect/google/check
```

- **CORS Policy (.env):**

```bash
CORS_ALLOW_ORIGIN='^https?://(localhost|127\.0\.0\.1)(:[0-9]+)?$'
```

---

## 🏗️ 3. Environment Configuration

### Database (Docker)

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

```bash
GOOGLE_CLIENT_ID=your_client_id_here
GOOGLE_CLIENT_SECRET=your_client_secret_here
```

---

## ⚙️ 4. System Configuration

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

## 🛡️ 5. Implementation: User Verification

The `UserChecker.php` handles pre-authentication logic, ensuring only active and verified users can log in.

| Condition           | Action                                         |
|--------------------|-----------------------------------------------|
| Account Inactive   | Throws Deactivated exception                  |
| Email Not Verified | Throws exception unless the provider is Google|

---

## 🎨 6. Frontend Integration

Place the following button in your Twig login template:

```twig
<div class="auth-wrapper">
    <a href="{{ path('connect_google_start') }}" id="google-login-link" class="btn-google">
        <ion-icon name="logo-google" class="icon-left"></ion-icon>
        Continue with Google
    </a>
</div>
```

---

## 🔧 Troubleshooting & Utilities

### Manual Database Permission Fix

If the Docker container denies access to `googleuser`, run these commands inside the container:

```bash
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