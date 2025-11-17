---
title: Lifecycle
---

## Application Life Cycle

### 1. Instantiation

First, you instantiate the `Slim\App` class using the `Slim\Builder\AppBuilder` class.
This is the Slim application object.
During instantiation, Slim registers default services for each application dependency.

Example:

```php
use Slim\Builder\AppBuilder;

$builder = new AppBuilder();
$app = $builder->build();
```

### 2. Add Middleware Stack

Add at least the `RoutingMiddleware` and `EndpointMiddleware` to handle to let Slim handle the routing and dispatching.

Example:

```php
use Slim\Middleware\EndpointMiddleware;
use Slim\Middleware\RoutingMiddleware;

// ...

$app->add(RoutingMiddleware::class);
$app->add(EndpointMiddleware::class);
```

### 3. Route Definitions

Define routes using the application instance's `get()`, `post()`, `put()`, `delete()`, `patch()`, `head()`, and
`options()` routing methods.
These instance methods register a route with the application's Router object.
Each routing method returns the Route instance so you can immediately invoke the Route instance's methods to add
middleware or assign a name.

Example:

```php
$app->get('/', function ($request, $response) {
    $response->getBody()->write('Hello, World!');

    return $response;
});
```

### 4. Application Runner

Invoke the application instance's `run()` method.
This method starts the following process:

```php
$app->run();
```

#### A. Enter Middleware Stack

The `run()` method begins to inwardly traverse the application's middleware stack.
This is a concentric structure of middleware layers that receive (and optionally manipulate) the Environment, Request,
and Response objects before (and after) the Slim application runs.
The Slim application is the innermost layer of the concentric middleware structure.
Each middleware layer is invoked inwardly beginning from the outermost layer.

#### B. Run Application

After the `run()` method reaches the innermost middleware layer, it invokes the application instance and dispatches the
current HTTP request to the appropriate application route object.
If a route matches the HTTP method and URI, the route's middleware and callable are invoked.
If a matching route is not found, the Not Found or Not Allowed handler is invoked.

#### C. Exit Middleware Stack

After the application dispatch process completes, each middleware layer reclaims control outwardly, beginning from the
innermost layer.

#### D. Send HTTP Response

After the outermost middleware layer cedes control, the application instance prepares, serializes, and returns the HTTP
response.
The HTTP response headers are set with PHP's native `header()` method, and the HTTP response body is output to the
current output buffer.
