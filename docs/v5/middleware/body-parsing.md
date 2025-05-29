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

* Reads `Content-Type` from request headers.
* Matches it with registered parsers.
* Falls back to structured syntax suffixes (RFC 6839), e.g. +json

## Supported media types

* application/json
* application/x-www-form-urlencoded
* application/xml
* text/xml