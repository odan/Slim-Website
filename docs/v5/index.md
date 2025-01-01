---
title: Slim 5 Documentation
---

<div class="alert alert-info">
    <p>
        This documentation is for <strong>Slim 5 (Alpha)</strong>. Looking for <a href="/docs/v4">Slim 4 Docs</a>?
    </p>
</div>

## Welcome

Slim is a lightweight PHP micro-framework designed for building fast and powerful web applications and APIs. 

At its core, Slim is a dispatcher that:

* Receives an HTTP request.
* Executes the appropriate callback.
* Returns an HTTP response.

That's it!

## Why use Slim?

Slim is perfect for creating APIs and web applications that are:

* **Simple yet effective:** Ideal for consuming, repurposing, or publishing data.
* **Flexible:** Suitable for rapid prototyping or building full-featured web apps with user interfaces.
* **Efficient:** Slim is fast and minimal, with very little overhead.

Unlike comprehensive frameworks like [Symfony](https://symfony.com/) or [Laravel](https://laravel.com/), 
Slim focuses on providing only the essential tools — no more, no less. 

This makes it ideal for projects where simplicity and speed matter most.

## How does Slim work?

Using Slim involves three key steps:

* **Set up a web server:** Use Nginx or Apache and configure it to route requests to a single "front-controller" PHP file.
* **Define routes:** Routes map specific HTTP requests to callback functions.
* **Run the app:** Instantiate, configure, and execute your Slim application.

## Example Application

Here's how a basic Slim app works:

```php
<?php

use Slim\Builder\AppBuilder;
use Slim\Middleware\BodyParsingMiddleware;
use Slim\Middleware\EndpointMiddleware;
use Slim\Middleware\ExceptionHandlingMiddleware;
use Slim\Middleware\ExceptionLoggingMiddleware;
use Slim\Middleware\RoutingMiddleware;

require __DIR__ . '/../vendor/autoload.php';

//  Instantiate App
$builder = new AppBuilder();
$app = $builder->build();

// Add middleware (First in, First out)
$app->add(RoutingMiddleware::class);
$app->add(BodyParsingMiddleware::class);
$app->add(ExceptionHandlingMiddleware::class);
$app->add(ExceptionLoggingMiddleware::class);
$app->add(EndpointMiddleware::class);

// Define app routes
$app->get('/', function ($request, $response, $args) {
    $response->getBody()->write('Hello, World!');
    
    return $response;
});

// Run app
$app->run();
```

Slim makes building web applications and APIs straightforward and fast. 

Start small, build big, and enjoy the simplicity of Slim!

## Request and response

In Slim, you frequently work with **Request** and **Response** objects. These objects represent:

* The HTTP request received by the web server.
* The HTTP response sent back to the client.

Each route in a Slim app receives the current Request and Response objects as arguments 
to its callback. These objects follow the [PSR-7 standard](/docs/v5/concepts/value-objects.html),
ensuring compatibility and flexibility.

### Key Points

* Routes can inspect and manipulate Request and Response objects as needed.
* Every route must return a PSR-7 Response object.

## Bring your own components

Slim is highly flexible and integrates seamlessly with other PHP components. 

You can enhance its functionality by adding:

* First-party components like:
  * [Slim-HttpCache](https://github.com/slimphp/Slim-HttpCache) for HTTP caching.
  * [PHP-View](https://github.com/slimphp/PHP-View) for PHP template rendering.
  * [Twig-View](https://github.com/slimphp/Twig-View) for Twig templating.
* Third-party libraries from [Packagist](https://packagist.org/).

This modular approach allows you to customize Slim to your project's needs.

## Navigating this documentation

### For Beginners:

Start from the beginning to understand Slim’s core concepts and architecture. This will provide a solid foundation before diving into more advanced topics.

### For Experienced Users:

Jump straight to the section that matches your current needs, such as:

* Request and Response handling
* Routing
* Error handling

This documentation is structured to guide you through Slim’s features, 
starting with its core principles and progressing to specific functionalities.

## Documentation License

<p style="text-align: left;">
    This website and documentation is licensed under the <a rel="license" href="https://opensource.org/licenses/MIT">MIT License</a>.
    <br />
    <a rel="license" href="https://opensource.org/license/MIT">
        <img alt="MIT License" style="border-width:0" src="https://img.shields.io/badge/License-MIT-yellow.svg" />
    </a>
</p>

