---
title: Web Servers
---

It is typical to use [the Front-Controller pattern](https://martinfowler.com/eaaCatalog/frontController.html) to funnel appropriate HTTP requests received by your web server to a single PHP file. 
The instructions below explain how to tell your web server to send HTTP requests to your PHP front-controller file.

## PHP built-in server

Run the following command in terminal to start localhost web server, assuming *./public/* is public-accessible directory with *index.php* file:

```bash
cd public/
php -S localhost:8888
```

If you are not using *index.php* as your entry point then change appropriately.

> **Warning:** The built-in web server was designed to aid application development. 
It may also be useful for testing purposes or for application demonstrations that are run in controlled environments. 
It is not intended to be a full-featured web server. 
It should not be used on a public network.

## Apache configuration

To run a Slim application on Apache, follow these steps to configure the server:

###  Step 1: Enable mod_rewrite

1. Ensure that the mod_rewrite module is installed and enabled. 
Use the following commands in the terminal:

```bash
sudo a2enmod rewrite
sudo a2enmod actions
```

2. Update Apache’s configuration to allow .htaccess to override settings:

* Open `/etc/apache2/apache2.conf` with root privileges.
* Modify the `<Directory>` directive for your project directory from `AllowOveride None` to `AllowOveride All` as shown below:

**Example**

```
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

### Step 2: Configure .htaccess

Place the `.htaccess` and `index.php` files in the same publicly accessible directory. 
Add the following rules to your `.htaccess` file:

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

### Step 3: Redirect Public Directory

To hide the `public/` directory in the URL, 
add another `.htaccess` file in the parent directory with the following rules:

```
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

These rules ensure proper internal redirection while maintaining clean URLs.

### Step 4: Restart Apache

After making changes, reload Apache to apply the new configuration. Use the following command:

```bash
sudo service apache2 restart
```

This works on most Debian/Ubuntu systems. For other Linux distributions, 
refer to your system’s documentation for restarting Apache.

### Step 5 (optional): Running in a sub-directory

If your Slim application runs in a sub-directory of the Apache document root, 
additional configuration is needed.

#### Option 1: Using the integrated BasePathMiddleware

The `BasePathMiddleware` automatically manages sub-directory paths.

Here’s an example:

```php
use Slim\Middleware\BasePathMiddleware;
use Slim\Middleware\EndpointMiddleware;
use Slim\Middleware\RoutingMiddleware;
// ...

$app->add(BasePathMiddleware::class);
$app->add(RoutingMiddleware::class);
$app->add(EndpointMiddleware::class);
// ...

$app->run();
```

**Note:** When using the `BasePathMiddleware`, 
ensure it is added before the `RoutingMiddleware` in your middleware stack.

#### Option 2: Using setBasePath() method

You can configure Slim to handle sub-directory URLs using one of the following methods

Set the base path for your Slim application to match the sub-directory.

Add the following to your application setup:

```php
$app->setBasePath('/myapp');
```

## Nginx configuration

This is an example Nginx virtual host configuration for the domain `example.com`.
It listens for inbound HTTP connections on port 80. It assumes a PHP-FPM server is running on port 9123. You should update the `server_name`, `error_log`, `access_log`, and `root` directives with your own values. 
The `root` directive is the path to your application's public document root directory; your Slim app's `index.php` front-controller file should be in this directory.

```bash
server {
    listen 80;
    server_name example.com;
    index index.php;
    error_log /path/to/example.error.log;
    access_log /path/to/example.access.log;
    root /path/to/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ \.php {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        fastcgi_pass 127.0.0.1:9123;
    }
}
```

## Caddy

The Caddy configuration is located in `/etc/caddy/Caddyfile`. Caddy requires `php-fpm` and have the FPM server running.
Assuming the FPM socket is at `/var/run/php/php-fpm.sock`, and your application is located in `/var/www`, the following configuration should work out of the box.

### HTTP configuration listening for any request

```bash
:80 {
        # Set-up the FCGI location
        php_fastcgi unix//var/run/php/php-fpm.sock
        # Set this path to your site's directory.
        root * /var/www/public
}
```

### HTTPS configuration with self-signed certificate

```bash
:443 {
        tls internal
        # Set-up the FCGI location
        php_fastcgi unix//var/run/php/php-fpm.sock
        # Set this path to your site's directory.
        root * /var/www/public
}
```

## IIS

Ensure the `Web.config` and `index.php` files are in the same public-accessible directory. The `Web.config` file should contain this code:

```bash
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="slim" patternSyntax="Wildcard">
                    <match url="*" />
                    <conditions>
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

## lighttpd

Your lighttpd configuration file should contain this code (along with other settings you may need). This code requires lighttpd >= 1.4.24.

```bash
url.rewrite-if-not-file = ("(.*)" => "/index.php/$0")
```

This assumes that Slim's `index.php` is in the root folder of your project (www root).

## Run From a Sub-Directory

If you want to run your Slim Application from a sub-directory in your Server's Root instead of creating a Virtual Host, you can configure `$app->setBasePath('/path-to-your-app');` right after the `AppFactory::create();`.
Assuming that your Server's Root is `/var/www/html/` and path to your Slim Application is `/var/www/html/my-slim-app` you can set the base path to `$app->setBasePath('/my-slim-app');`.

```php
<?php

use Slim\Factory\AppFactory;
// ...

$app = AppFactory::create();
$app->setBasePath('/my-slim-app');

// ...

$app->run();
```

