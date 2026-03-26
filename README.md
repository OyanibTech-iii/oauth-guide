
URI's 
http://127.0.0.1:8000
http://127.0.0.1:8000/connect/google/check
```bash

-----------------------------------------------------------------
composer require knpuniversity/oauth2-client-bundle
composer require league/oauth2-google
composer require symfonycasts/verify-email-bundle
-----------------------------------------------------------------
```

configure your account on https://console.cloud.google.com/auth/clients

CORS_ALLOW_ORIGIN='^https?://(localhost|127\.0\.0\.1)(:[0-9]+)?$'
Client ID
Client secret
-----------------------------------------------------------------

.env line 16
```bash

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

-----------------------------------------------------------------
```bash

knpu_oauth2_client:
  clients:
    google:
      type: google
      client_id: '%env(GOOGLE_CLIENT_ID)%'
      client_secret: '%env(GOOGLE_CLIENT_SECRET)%'
      # This 'route' must generate the URI you put in Google Console
      redirect_route: connect_google_check
      redirect_params: {}
 ```

------------------------------------------------------------------
security.yaml
```bash

            custom_authenticators:
              - App\Security\LoginFormAuthenticator
              - App\Security\GoogleAuthenticator
            user_checker: App\Security\UserChecker
```

------------------------------------------------------------------
```bash

security/UserChecker.php
<?php

namespace App\Security;

use App\Entity\User;
use Symfony\Component\Security\Core\Exception\CustomUserMessageAuthenticationException;
use Symfony\Component\Security\Core\User\UserCheckerInterface;
use Symfony\Component\Security\Core\User\UserInterface;

class UserChecker implements UserCheckerInterface
{
    public function checkPreAuth(UserInterface $user): void
    {
        if (!$user instanceof User) {
            return;
        }

        // Global check: If inactive, stop authentication immediately
        if (method_exists($user, 'isActive') && !$user->isActive()) {
            throw new CustomUserMessageAuthenticationException(
                'Your account has been deactivated. Please contact support.'
            );
        }
        if (
            method_exists($user, 'isVerified')
            && !$user->isVerified()
            && method_exists($user, 'getProvider')
            && $user->getProvider() !== 'google'
        ) {
            throw new CustomUserMessageAuthenticationException(
                'Verified Account is Only Allowed to Login. Make sure to verify your email address before logging in.'
            );
        }

    }

    public function checkPostAuth(UserInterface $user): void
    {
        // This runs after the password has been checked. 
        // You can leave it empty or add additional checks here.
    }
}
```

------------------------------------------------------------------
  templates/login
  ```bash

  <div class="auth-wrapper">
        <a href="{{ path('connect_google_start') }}" id="google-login-link" class="btn-google">
            <ion-icon name="logo-google" class="icon-left"></ion-icon>
            Continue with Google
        </a>
    </div>
```

------------------------------------------------------------------
mysql/bypass 
```bash

-- Create the user if it doesn't exist for that specific IP range
CREATE USER IF NOT EXISTS 'googleuser'@'%' IDENTIFIED BY 'your_password_here';

-- Grant all privileges on your database
GRANT ALL PRIVILEGES ON your_database_name.* TO 'googleuser'@'%';

-- Refresh the privileges
FLUSH PRIVILEGES;
```

------------------------------------------------------------------
clear git commit
```bash

git reset --soft HEAD~1

```