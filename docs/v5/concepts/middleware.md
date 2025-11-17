~~---
title: Middleware
---

Middleware is a way to run code before and after your Slim app processes a request.
Think of it as a set of layers that wrap around your app,
giving you full control to modify requests and responses.

This is handy for things like: Authentication, Authorization, Logging and Error handling.

## How middleware works

Middleware is like a set of "checkpoints" in your app.
Every HTTP request passes through them on its way to the core Slim application,
and the response takes the same route back out.

Slim 5 uses a **FIFO (First In, First Out)** approach for middleware. Here’s how it works:

* When a Request comes in, it passes through the middleware in the **same order they were added** (first one added, first to run).
* Once the Request reaches the route handler (your core logic), the app processes it and generates a Response.
* The Response travels back out through the middleware in the reversed order, getting modified or processed along the way.

```
Request  → [Middleware #1] → [Middleware #2] → [Your Core App] ↓
Response ← [Middleware #1] ← [Middleware #2] ←
```

The middleware that’s added first is the one that processes both the Request and the Response first.

### Why FIFO?

Using **FIFO** makes it super clear what’s happening.
The middleware runs in **the exact order you added it**,
for incoming requests.

This lets you manage tasks in logical steps:

* **First Middleware:** Handles early concerns like authentication.
* **Middle Layers:** Process logging or request formatting.
* **Last Middleware:** Handles post-processing like CSRF checks or response tweaks.

## Adding middleware

Adding middleware in Slim is super simple. You just stack it using the App `add()` or `addMiddleware()` method.

```php
// Add middleware using the class syntax (for dependency injection)
$app->add(ExampleMiddleware::class);

// Add middleware by creating the object manually
$app->add(new ExampleMiddleware());
```

Here's an example:

```php
<?php

use Slim\Builder\AppBuilder;
use Slim\Middleware\EndpointMiddleware;
use Slim\Middleware\RoutingMiddleware;

$builder = new AppBuilder();
$app = $builder->build();

$app->add(RoutingMiddleware::class);  // First layer
$app->add(EndpointMiddleware::class); // Second layer

$app->get('/', function ($request, $response) {
    $response->getBody()->write('Hello, World!');

    return $response;
});

$app->run();
```

Example using the `addMiddleware()` method:

```php
$app->addMiddleware(new AuthMiddleware());      // First layer
$app->addMiddleware(new LoggingMiddleware());   // Second layer
$app->addMiddleware(new CsrfMiddleware());      // Third layer
```

**Note:** It is recommended to use the `add()` method by default because of the DI container integration.

Depending on your use case, you can attach middleware globally to the entire app,
or locally to specific **routes** or **route groups**.

```php
// Add middleware to a specific route using dependency injection
$app->get('/', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Route logic here
})->add(ExampleMiddleware::class);

// Add middleware to a route group using dependency injection
$app->group('/api', function (RouteGroup $group) {
    // Define routes here
})->add(ExampleMiddleware::class);

```

## Middleware execution order

Slim processes middleware in a **First In, First Out (FIFO)** order.

This means the first middleware you add is the first to be executed for both requests and responses.

**Request Flow:**

* Middleware is executed in the order it’s added.
* The request passes through each middleware, eventually reaching the core Slim application.

**Response Flow:**
* The response flows back through the middleware in the same order, with each middleware getting a chance to modify it.

**Example:**

```php
$app->add(MiddlewareOne::class);
$app->add(MiddlewareTwo::class);
$app->add(MiddlewareThree::class);
```

In this case:

* **Request Flow:** `MiddlewareOne` runs first, followed by `MiddlewareTwo`, then `MiddlewareThree`.
* **Response Flow:** The response flows back in the order - `MiddlewareThree`, `MiddlewareTwo`, then `MiddlewareOne`.

## Creating custom middleware

Middleware is essentially a callable that accepts two arguments:

* A Request object: `ServerRequestInterface`
* A RequestHandler object: `RequestHandlerInterface`.

Every middleware **MUST** return an instance of `Psr\Http\Message\ResponseInterface`.
This makes it compatible with the Slim middleware pipeline and **PSR-15** standards.

**What is PSR-15 middleware?**

[PSR-15](https://www.php-fig.org/psr/psr-15/) defines standard interfaces for handling HTTP requests and middleware.
Slim fully supports PSR-15 middleware, enabling smooth integration of both custom
and third-party components.

### Writing a Simple Middleware Class

Here is a basic example of a PSR-15 middleware:

```php
<?php

namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

final class ExampleMiddleware implements MiddlewareInterface
{
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // Optional: Handle the incoming request
        // ...

        // Pass the request to the next middleware or handler
        $response = $handler->handle($request);

        // Optional: Handle the outgoing response
        // ...

        return $response;
    }
}

```

In this example:

* You can authenticate, log, or modify requests before they reach the Slim app.
* You can log, transform, or enhance responses before they’re sent back to the client.

### Creating a new response in a Middleware

To create a custom response, use the `ResponseFactoryInterface`,
which provides the `createResponse()` method.

Here is an example:

```php
<?php

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

final class CustomResponseMiddleware implements MiddlewareInterface
{
    private ResponseFactoryInterface $responseFactory;

    public function __construct(ResponseFactoryInterface $responseFactory)
    {
        $this->responseFactory = $responseFactory;
    }

    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // Example: Return a new response if a condition is met
        if (/* some condition */) {
            $response = $this->responseFactory->createResponse(403);
            $response->getBody()->write('Forbidden');

            return $response;
        }

        // Otherwise, pass the request to the next middleware
        return $handler->handle($request);
    }
}

```

**Note::** New responses are `200 OK` by default.

It is possible to provide a custom status code like this:

```php
$response = $this->responseFactory->createResponse(201);
```

### Invokable class middleware

Middleware can also be an **invokable class**, which uses the magic `__invoke()` method.

Here is an example:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;

final class ExampleMiddleware
{
    public function __invoke(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // Process the incoming request
        // ...

        // Pass to the next middleware
        $response = $handler->handle($request);

        // Optionally modify the outgoing response
        // ...

        return $response;
    }
}

```

This approach works well for simple, lightweight middleware.

### Closure middleware

For quick and simple tasks, you can use a closure as middleware. Here is an example:

```php
<?php

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;

// ...

$app->add(function (ServerRequestInterface $request, RequestHandlerInterface $handler) {
    // Check for an "Authorization" header
    $auth = $request->getHeaderLine('Authorization');
    if (!$auth) {
        // Return a 401 Unauthorized response
        $response = $this->get(ResponseFactoryInterface::class)->createResponse();
        $response->getBody()->write('Unauthorized');

        return $response;
    }

    // Pass to the next middleware
    return $handler->handle($request);
});

// ...
$app->run();

```

Closures are great for quick middleware logic.
For anything more complex, it’s better to use a class.

### Route middleware

Route middleware is invoked _only if_ its route matches the current HTTP request method and URI.
Route middleware is specified immediately after you invoke any of the Slim application's routing methods (e.g., **get()** or **post()**).
Each routing method returns an instance of **\Slim\Route**, and this class provides the same middleware interface as the Slim application instance.
Add middleware to a Route with the Route instance's **add()** method.
This example adds the Closure middleware example above:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Slim\Builder\AppBuilder;
use Slim\Middleware\EndpointMiddleware;
use Slim\Middleware\RoutingMiddleware;

$builder = new AppBuilder();
$app = $builder->build();

$app->add(RoutingMiddleware::class);
$app->add(EndpointMiddleware::class);

$middleware = function (ServerRequestInterface $request, RequestHandler $handler) {
    $response = $handler->handle($request);
    $response->getBody()->write('World');

    return $response;
};

$app->get('/', function (ServerRequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello ');

    return $response;
})->add($middleware);

$app->run();
```

This would output this HTTP response body:

```bash
Hello World
```

### Group middleware

Middleware can be applied not only to individual routes or the overall application
but also to **route groups**. This is useful when multiple routes share common logic,
like authentication or response formatting.

Here is a sample application with middleware applied to a group of routes:

```php
<?php

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Slim\Builder\AppBuilder;
use Slim\Routing\RouteGroup;
// ...

$builder = new AppBuilder();
$app = $builder->build();

// ...

// A basic root route
$app->get('/', function (ServerRequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello World');

    return $response;
});

// Define a group of routes under '/utils'
$app->group('/utils', function (RouteGroup $group) {
    $group->get('/date', function (ServerRequestInterface $request, ResponseInterface $response) {
        $response->getBody()->write(date('Y-m-d H:i:s'));

        return $response;
    });

    $group->get('/time', function (ServerRequestInterface $request, ResponseInterface $response) {
        $response->getBody()->write((string)time());

        return $response;
    });
})->add(function (ServerRequestInterface $request, RequestHandlerInterface $handler) {
    // Middleware for the entire group
    $response = $handler->handle($request);
    $dateOrTime = (string) $response->getBody();

    $response = $this->get(ResponseFactoryInterface::class)->createResponse();
    $response->getBody()->write('It is now ' . $dateOrTime . '. Enjoy!');

    return $response;
});

$app->run();
```

When calling the **/utils/date** method, this would output a string similar to the below.

```
It is now 2015-07-06 03:11:01. Enjoy!
```

Visiting **/utils/time** would output a string similar to the below.

```
It is now 1436148762. Enjoy!
```

But visiting **/** *(domain-root)*, would be expected to generate the following output as no middleware has been assigned.

```
Hello World
```

**Note:** It's recommended to define a class for better dependency injection support.

```php
$app->group('/api', function (RouteGroup $group) {
    // ...
})->add(MyApiMiddleware::class);
```

### Passing variables from middleware

The simplest way to pass data from middleware to a route is by using the request **attributes**.
These attributes allow you to attach custom data to the request,
making it accessible further down the pipeline.

The easiest way to pass attributes from middleware is to use the request's attributes.

Setting an attribute:

```php
$request = $request->withAttribute('foo', 'bar');
```

Reading an attribute:

```php
$foo = $request->getAttribute('foo');
```

### Quick tips for middleware

* **Keep it simple:** Each middleware should do one thing really well.
* **Short-circuit when necessary:** If a middleware detects an issue (like invalid credentials), it can immediately return a response and skip the rest of the stack.
* **Use libraries:** Many common tasks, like CSRF or authentication, may have pre-built middleware you can use.
* **Be consistent:** Add middleware in the logical order they need to run. Remember, first in, first out.

## Finding available middleware

Before writing your own, consider checking if a PSR-15 middleware class already exists
to meet your needs. Many commonly used middleware components are available online.

Here are some resources to help you search:

* [Github PSR-15: HTTP Server Request Handlers](https://github.com/topics/psr-15)
* [middlewares/awesome-psr15-middlewares](https://github.com/middlewares/awesome-psr15-middlewares)
