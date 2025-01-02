---
title: Templates
---

Slim doesn’t include a traditional view layer like most MVC frameworks. 
Instead, Slim treats the HTTP response itself as the "view." 
Each route in your application is responsible for preparing and returning 
an appropriate **PSR-7 Response** object.

> Slim's "view" is the HTTP response.

## Rendering Templates

While Slim does not dictate a specific template engine, 
it offers integrations with popular options to simplify rendering templates:

* [Twig-View](twig-view.html): Use the Twig template engine for flexible, powerful, and secure templates.
* [PHP-View](php-view.html): Use plain PHP templates with minimal overhead.

Both components are lightweight, easy to integrate, and provide convenient ways to write rendered output to the Response object.

## Using other Template systems

Slim’s flexibility allows you to use any PHP-based template engine or custom rendering approach.
The only requirement is that you write the rendered template output to the 
body of the **PSR-7 Response** object.

### General Workflow for Template Rendering

* Prepare your data (e.g., query results, configuration values).
* Render the template using your chosen engine.
* Write the rendered output to the PSR-7 Response body.

Slim’s flexibility means you’re free to use the templating approach that works best for your project—whether 
that’s Twig, PHP templates, or a custom solution.

Here’s a basic example showing how to work with any template engine:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

$app->get('/example', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Example: Render a custom template
    $output = MyCustomTemplateEngine::render('template-file', ['key' => 'value']);
    
    // Write output to the response body
    $response->getBody()->write($output);
    
    return $response;
});

```