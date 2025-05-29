---
title: Body Parsing Middleware
---

Handling JSON, XML, or form data is a common requirement in web APIs. 
While PSR-7 does not provide body parsing out of the box, 
Slim 5 includes BodyParsingMiddleware to simplify this task.

## Installation

Add the middleware after `ExceptionMiddleware` in your middleware stack:

```php
<?php

use Slim\Builder\AppBuilder;
use Slim\Middleware\BodyParsingMiddleware;
use Slim\Middleware\EndpointMiddleware;
use Slim\Middleware\ExceptionMiddleware;
use Slim\Middleware\RoutingMiddleware;

$builder = new AppBuilder();
// ...
$app = $builder->build();

$app->add(ExceptionMiddleware::class);
$app->add(BodyParsingMiddleware::class); // <--- Place the middleware here
$app->add(RoutingMiddleware::class);
$app->add(EndpointMiddleware::class);

// ...

$app->run();
```

## Configuration

Define `BodyParsingMiddleware` in your dependency container:

```php
use Psr\Container\ContainerInterface;
use Slim\Media\MediaTypeDetector;
use Slim\Middleware\BodyParsingMiddleware;

return [

    BodyParsingMiddleware::class => function (ContainerInterface $container) {
        $mediaTypeDetector = $container->get(MediaTypeDetector::class);
        $middleware = new BodyParsingMiddleware($mediaTypeDetector);

        $middleware = $middleware
            ->withDefaultBodyParsers()
            ->withDefaultMediaType('application/json');

        return $middleware;
    },
];

```

## Usage

Use `$request->getParsedBody()` to access parsed data.

You don’t need to manually decode request body, because this middleware
automatically handles JSON, form, or XML data.

```php
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

$app->post('/', function (ServerRequestInterface $request, ResponseInterface $response): ResponseInterface {
    $data = $request->getParsedBody();
    
    $html = var_export($data, true);
    $response->getBody()->write($html);
    
    return $response;
});
```

## Media type detection

BodyParsingMiddleware detects and parses body content as follows:

* Prioritizes, the `Accept` request header.
* Fallback to `Content-Type` if `Accept` is empty or missing
* Matches it with registered parsers.

**Important Note**

Structured suffix media types (e.g. application/vnd.api+json) are not matched unless
you manually register a handler for the specific type.

## Registers standard handlers for these media types:

The `withDefaultBodyParsers` method registers the standard handlers for all supported media types.

```php
$middleware = (new BodyParsingMiddleware($mediaTypeDetector))
    ->withDefaultBodyParsers();
```

## Custom Media Types

The `withBodyParser` method registers a custom parser for a given media type.

**Example: Register a YAML body parser**

This lets the middleware handle requests with Content-Type: `application/x-yaml`

```php

use Symfony\Component\Yaml\Yaml;
// ...

$middleware = (new BodyParsingMiddleware($mediaTypeDetector))
    ->withBodyParser('application/x-yaml', function (string $input) {
        return Yaml::parse($input);
    });
```

**Example: Support structured media type**

This allows clients to send content with Content-Type: `application/vnd.api+json` and have it parsed correctly.

```php
$middleware = (new BodyParsingMiddleware($mediaTypeDetector))
    ->withBodyParser('application/vnd.api+json', function (string $input) {
        $data = json_decode($input, true);
        return is_array($data) ? $data : null;
    });
```

Sets the media type to fall back to when no Content-Type or Accept header is provided.

Example: Fallback to application/json

```php
$middleware = (new BodyParsingMiddleware($mediaTypeDetector))
    ->withDefaultMediaType('application/json');
```

## Supported media types

* application/json
* application/x-www-form-urlencoded
* application/xml
* text/xml