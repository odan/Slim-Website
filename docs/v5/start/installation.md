---
title: Installation
---

## System Requirements

* **Web server:** Ensure URL rewriting is enabled.
* **PHP Version:** PHP 8.2 or newer.

## Step 1: Install Composer

Composer is required to manage Slim and its dependencies. 
If you don’t have it installed, follow the [Composer installation guide](https://getcomposer.org/download/).

## Step 2: Install Slim Framework

To install Slim, use Composer. Navigate to your project’s root directory 
and run the following command:

```bash
composer require slim/slim:5.x-dev
```

This will download Slim and its dependencies into the `vendor/` directory.

## Step 3: Install a PSR-7 Implementation

Slim requires a PSR-7 implementation. Choose the one that best suits your project 
and install it using Composer.

### Options:

1. [Slim PSR-7](https://github.com/slimphp/Slim-Psr7)

```bash
composer require slim/psr7
```

2. [Nyholm PSR-7](https://github.com/Nyholm/psr7) and [Nyholm PSR-7 Server](https://github.com/Nyholm/psr7-server)

```bash
composer require nyholm/psr7 nyholm/psr7-server
```

3. [Guzzle PSR-7](https://github.com/guzzle/psr7)

```bash
composer require guzzlehttp/psr7
```

4. [Laminas Diactoros](https://github.com/laminas/laminas-diactoros)

```bash
composer require laminas/laminas-diactoros
```

## Step 4: Install a PSR-11 Container Implementation

To manage dependencies in your Slim application, 
install a PSR-11 container implementation, such as [PHP-DI](https://php-di.org/):

```bash
composer require php-di/php-di
```

## Step 5: Create a "Hello World" Application

Set up a basic Slim app by creating a file named `public/index.php` with the following content:

```php
<?php

use Slim\Builder\AppBuilder;
use Slim\Middleware\BodyParsingMiddleware;
use Slim\Middleware\ExceptionHandlingMiddleware;
use Slim\Middleware\ExceptionLoggingMiddleware;
use Slim\Middleware\RoutingMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$builder = new AppBuilder();
$app = $builder->build();

// Add middleware (First in, First out)
$app->add(RoutingMiddleware::class);
$app->add(BodyParsingMiddleware::class);
$app->add(ExceptionHandlingMiddleware::class);
$app->add(EndpointMiddleware::class);

// Define app routes
$app->get('/', function ($request, $response, $args) {
    $response->getBody()->write('Hello, World!');
    
    return $response;
});

// Run app
$app->run();
```
