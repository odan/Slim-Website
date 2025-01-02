---
title: Deployment
---

Congratulations! If you've made it this far, you've built something awesome with Slim. 
But before you celebrate, it’s time to deploy your application to a production server.

There are many ways to do this that are beyond the scope of this documentation. 
In this section, we provide some notes for various set-ups.

### Disable error display in production

In production, it's critical to prevent detailed error information from being 
shown to users. Slim makes this easy with the `ExceptionHandler` configuration.

Use settings:

```php
// ...

$app->setSettings(
    [
        'display_error_details' => false,
    ]
);

```

or

```php

use Slim\Interfaces\ExceptionHandlerInterface;
// ...

$exceptionHandler = $exceptionHandler->withDisplayErrorDetails(false);
```

### PHP configuration

Ensure your PHP installation is also configured to not display errors. 

**php.ini**

```ini
display_errors = 0
```

## Build your application for production

### Optimize dependencies

Run Composer in production mode to install only the required dependencies:

```bash
composer install --no-dev --optimize-autoloader
```

### Protect sensitive data

Avoid hardcoding sensitive data (like database credentials) in your code. 
Use environment variables and a library like vlucas/phpdotenv to manage them:

```
APP_ENV=production
DATABASE_URL=mysql://user:password@localhost/dbname
```

Load these variables in your application:

```php
$dotenv = Dotenv::createImmutable(__DIR__ . '/../');
$dotenv->load();

$appEnv = $_ENV['APP_ENV'] ?? 'development';
```

## Deploying to a Modern Hosting Platform

### Containerization with Docker

Package your application into a Docker container for consistency across environments:

```dockerfile
FROM php:8.2-fpm

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    libzip-dev zip unzip git curl && \
    docker-php-ext-install zip pdo pdo_mysql

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Copy application
COPY . /var/www

# Install dependencies
RUN composer install --no-dev --optimize-autoloader

# Expose port 9000 and run PHP-FPM
EXPOSE 9000
CMD ["php-fpm"]
```

docker-compose.yml:

```yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "9000:9000"
    volumes:
      - .:/var/www
    environment:
      APP_ENV: production

```

Run the application:

```
docker-compose up -d
```

## Deploy to cloud platforms

Modern platforms like AWS, Google Cloud, DigitalOcean, 
or Heroku support PHP applications with minimal setup. 

For serverless deployments, consider [Laravel Vapor](https://vapor.laravel.com/) (works with Slim) or AWS Lambda.

## Automate deployment

Automating your deployment ensures consistency and reduces downtime. 

Use modern tools for deployment workflows:

* GitHub Actions: Automate builds, tests, and deployments directly from your repository.
* CircleCI/Travis CI: Integrate continuous deployment pipelines for more advanced setups.
* Deployer: A PHP deployment tool tailored for PHP applications.

Example: Using GitHub Actions:

**.github/workflows/deploy.yml:**

```
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: 8.2

      - name: Install Dependencies
        run: composer install --no-dev --optimize-autoloader

      - name: Deploy Application
        run: scp -r . user@yourserver:/var/www/html

```

## Additional notes

* **Review Your Web Server Configuration:** Ensure your web server (e.g., Nginx or Apache) is properly set up to serve your application. See the Web Servers Documentation for details.
* **Enable Caching:** Enable OPcache (PHP) and use caching (middleware) to improve the performance 
* **Monitor Your Application:** Use tools like New Relic or Sentry for monitoring and error tracking in production.

