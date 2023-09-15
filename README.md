# Deal with Request-ID and Correlation-ID in Laravel applications

![bilfeldt/laravel-correlation-id](art/banner.png)

[![Latest Version on Packagist](https://img.shields.io/packagist/v/bilfeldt/laravel-correlation-id.svg?style=flat-square)](https://packagist.org/packages/bilfeldt/laravel-correlation-id)
[![GitHub Tests Action Status](https://img.shields.io/github/workflow/status/bilfeldt/laravel-correlation-id/run-tests?label=tests)](https://github.com/bilfeldt/laravel-correlation-id/actions?query=workflow%3Arun-tests+branch%3Amain)
[![StyleCI Code Style Status](https://github.styleci.io/repos/424928370/shield)](https://github.styleci.io/repos/424928370/shield)
[![Total Downloads](https://img.shields.io/packagist/dt/bilfeldt/laravel-correlation-id.svg?style=flat-square)](https://packagist.org/packages/bilfeldt/laravel-correlation-id)

Create request Correlation-IDs via middleware and pass both this globally unique `Correlation-ID` and any user provided `Request-ID` to the global log context.

| Version | Laravel     | PHP                     |
|---------|-------------|-------------------------|
| 1.*     | 8.* \| 9.*  | 7.4.* \| 8.0.* \| 8.1.* |
| 2.*     | 10.*        | 8.1.* \| 8.2.*          |

## Motivation

Each and every request should have a unique _Correlation ID_ which uniquely determines the user interaction. This _Correlation ID_ should then be added to all log entries as context, to error reporting and to any subsystem like external API calls or queued jobs. Doing so, makes is possible to track relevant touch points for a given user request. Read more [here](https://microsoft.github.io/code-with-engineering-playbook/observability/correlation-id/).

A client can also provide a _Request ID_ which it is good practice to return again in the response and correlate to the _Correlation ID_. Read more [here](https://http.dev/x-request-id).

## Installation

You can install the package via composer:

```bash
composer require bilfeldt/laravel-correlation-id
```

### Assigning Correlation ID to requests

> [!NOTE]
> Ideally, the unique Correlation ID should be created at the **very first touch point** of your infrastructure like the initial server which could be a load balancer

As it can be tricky to create _Correlation ID_ on the server level and this is so easy in Laravel using middleware. This package provides a middleware for creating a _Correlation ID_ and attaching it to the request as a header `Correlation-ID` and to the response header as well. You should either assign the correlation id on the first touch point of your infrastructure or register the `CorrelationIdMiddleware` middleware globally as the first middleware in the `$middleware` property of your `app/Http/Kernel.php` class:

```php
// app/Http/Kernel.php

/**
 * The application's global HTTP middleware stack.
 *
 * These middleware are run during every request to your application.
 *
 * @var array<int, class-string|string>
 */
protected $middleware = [
    \Bilfeldt\Middleware\CorrelationIdMiddleware::class, // <!-- Add this globally as the first toutchpoint
    // \App\Http\Middleware\TrustHosts::class,
    \App\Http\Middleware\TrustProxies::class,
    \Illuminate\Http\Middleware\HandleCors::class,
    \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
];
```

### Assigning Request ID to responses

If the client provides a _Request ID_ in the request header, then it is good practice to copy that to the response header. This is done by adding the `RequestIdMiddleware` middleware globally in the `$middleware` property of your `app/Http/Kernel.php` class:

```php
// app/Http/Kernel.php

/**
 * The application's global HTTP middleware stack.
 *
 * These middleware are run during every request to your application.
 *
 * @var array<int, class-string|string>
 */
protected $middleware = [
    \Bilfeldt\Middleware\CorrelationIdMiddleware::class,
    \Bilfeldt\Middleware\RequestIdMiddleware::class, // <!-- Add this globally
    // \App\Http\Middleware\TrustHosts::class,
    \App\Http\Middleware\TrustProxies::class,
    \Illuminate\Http\Middleware\HandleCors::class,
    \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
];
```

## Usage

This package registers a few macros on the `Illuminate\Http\Request` class:

```php
$request->getCorrelationId() // UUID or null if not assigned

$request->getClientRequestId() // The `Request-ID` provided by the client in the header `Request-ID` if any is provided
```

### Add global log context

It is possible to add both the _Correlation ID_ and the _Request ID_  to the global log context so that any writing to logs once those are added will automatically have the id's attached as context.

It is recommended to add them to the logging context just after assigning them to the request. This can be done by globally registering the middleware `LogContextMiddleware` in the `$middleware` property of your `app/Http/Kernel.php` class:

```php
// app/Http/Kernel.php

/**
 * The application's global HTTP middleware stack.
 *
 * These middleware are run during every request to your application.
 *
 * @var array<int, class-string|string>
 */
protected $middleware = [
    \Bilfeldt\Middleware\CorrelationIdMiddleware::class,
    \Bilfeldt\Middleware\RequestIdMiddleware::class,
    \Bilfeldt\Middleware\LogContextMiddleware::class, // <!-- Add this globally AFTER assigning Correlation ID and Request ID.
    // \App\Http\Middleware\TrustHosts::class,
    \App\Http\Middleware\TrustProxies::class,
    \Illuminate\Http\Middleware\HandleCors::class,
    \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
];
```

### Passing Correlation ID and Request ID to queued jobs

Any queued job that is dispatched during a request will be picked up by another process. Hence the `Request` object will not have the Correlation ID per default.

TODO: HOW CAN WE PASS THE CORRELATION ID AND REQUEST ID TO THE QUEUED JOBS AUTOMATICALLY DURING DISPATCH?

TODO: HOW CAN WE AUTOMATICALLY FETCH THEM WHEN A QUEUED JOB IS BEING HANDLED AND POPULATE THEM INTO THE REQUEST OBJECT?


## Testing

```bash
composer test
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](.github/CONTRIBUTING.md) for details.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [Anders Bilfeldt](https://github.com/bilfeldt)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.