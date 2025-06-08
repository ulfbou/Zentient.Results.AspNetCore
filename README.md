> **⚠️ Deprecated:**  
> This repository is no longer maintained. Please use the [Zentient.Endpoints](https://github.com/ulfbou/Zentient.Endpoints) repository for future development and updates.

# Zentient.Results.AspNetCore

[![NuGet Version](https://img.shields.io/nuget/v/Zentient.Results.AspNetCore.svg?style=flat-square)](https://www.nuget.org/packages/Zentient.Results.AspNetCore/)
[![NuGet Downloads](https://img.shields.io/nuget/dt/Zentient.Results.AspNetCore.svg?style=flat-square)](https://www.nuget.org/packages/Zentient.Results.AspNetCore/)
[![.NET](https://github.com/ulfbou/Zentient.Results.AspNetCore/actions/workflows/dotnet.yml/badge.svg)](https://github.com/YourOrg/Zentient.Results.AspNetCore/actions/workflows/dotnet.yml) ## Build Robust, Explicit APIs in ASP.NET Core with Functional Results

Zentient.Results.AspNetCore is a powerful integration library that brings the simplicity and clarity of the [Zentient.Results](https://github.com/ulfbou/Zentient.Results) functional result type directly into your ASP.NET Core applications. Say goodbye to unhandled exceptions and inconsistent error responses. This library helps you build highly predictable, maintainable, and developer-friendly web APIs by explicitly handling operation outcomes.

Leveraging the industry-standard [RFC 7807 Problem Details](https://datatracker.ietf.org/doc/html/rfc7807) specification, Zentient.Results.AspNetCore automatically transforms your `Zentient.Results.IResult` and `Zentient.Results.IResult<T>` instances into well-structured HTTP responses, ensuring your API communicates success or failure with precision and consistency.

## Why Use Zentient.Results.AspNetCore?

* **Explicit Error Handling**: Move beyond throwing exceptions. Return `IResult` to clearly communicate operation success or specific failures, making your code easier to read, test, and debug.
* **Consistent API Responses**: Automatically maps `IResult` failures to RFC 7807 `ProblemDetails` or `ValidationProblemDetails`, ensuring all your error responses follow a uniform, standardized format.
* **Reduced Boilerplate**: Integrate Zentient.Results seamlessly into your ASP.NET Core MVC controllers and Minimal APIs with global filters and conventions, minimizing repetitive error-handling code.
* **Improved Developer Experience**: Provide API consumers with rich, machine-readable error details, including custom error codes, categories, and trace IDs, enhancing client-side error handling.
* **Promotes Best Practices**: Encourages the adoption of functional programming patterns like Railway-Oriented Programming within your ASP.NET Core application layer.

## Features at a Glance

* **Automatic Problem Details Conversion**: Seamlessly converts `IResult` failures to `application/problem+json` responses.
* **Customizable Problem Types**: Generates Problem Details `type` URIs based on `ErrorCategory` or custom error codes.
* **MVC & Minimal API Support**: Global `ProblemDetailsResultFilter` for MVC and `ZentientResultEndpointFilter` for Minimal APIs.
* **Enhanced Validation Handling**: Overrides `InvalidModelStateResponseFactory` to emit `ValidationProblemDetails` for model state errors.
* **Trace ID Integration**: Automatically includes `traceId` in Problem Details extensions for easier distributed tracing.
* **Configurable Options**: Fine-tune Problem Details behavior and Zentient-specific options via dependency injection.
* **Intelligent Status Mapping**: Opinionated mapping of `Zentient.Results.ErrorCategory` to appropriate HTTP status codes (e.g., `NotFound` -> 404, `Validation` -> 400).

## Quick Start

1.  **Install the NuGet package**:

    ```bash
    dotnet add package Zentient.Results.AspNetCore - - version 0.1.0
    ```

2.  **Add services in `Program.cs`**:

    ```csharp
    // Program.cs
    using Zentient.Results;
    using Zentient.Results.AspNetCore;

    var builder = WebApplication.CreateBuilder(args);

    builder.Services.AddControllers(options =>
    {
        // Add this line for MVC controllers
        options.Filters.AddService<ProblemDetailsResultFilter>();
    });

    builder.Services.AddZentientResultsAspNetCore(
        configureZentientProblemDetails: options =>
        {
            // Optional: Customize the base URI for problem types
            options.ProblemTypeBaseUri = "[https://yourdomain.com/errors/](https://yourdomain.com/errors/)";
        });

    var app = builder.Build();

    // For Minimal APIs, register the endpoint filter
    app.UseZentientResultsForMinimalApi(); // Global filter for Minimal APIs

    app.MapControllers();
    app.Run();
    ```

3.  **Return `IResult` from your API endpoints**:

    ```csharp
    // Example Controller Action
    using Microsoft.AspNetCore.Mvc;
    using Zentient.Results;

    [ApiController]
    [Route("[controller]")]
    public class ProductsController : ControllerBase
    {
        [HttpGet("{id}")]
        public IActionResult GetProduct(int id)
        {
            IResult<Product> result = GetProductFromService(id); // Returns IResult<Product>

            // The filter will automatically convert 'result' to appropriate HTTP response
            return result.ToActionResult(); // Explicit conversion also available
        }

        private IResult<Product> GetProductFromService(int id)
        {
            if (id == 42)
            {
                return Result.Success(new Product(id, "Zentient Widget"));
            }
            if (id < 0)
            {
                return Result.Validation<Product>(
                    new ErrorInfo(ErrorCategory.Validation, "INVALID_ID", "Product ID must be positive."));
            }
            return Result.Failure<Product>(
                new ErrorInfo(ErrorCategory.NotFound, "PRODUCT_NOT_FOUND", $"Product {id} not found."));
        }
    }

    public record Product(int Id, string Name);
    ```

    * If `GetProductFromService(42)` is called, it returns `HTTP 200 OK` with `{"id":42, "name":"Zentient Widget"}`.
    * If `GetProductFromService(-1)` is called, it returns `HTTP 400 Bad Request` with `ValidationProblemDetails`.
    * If `GetProductFromService(99)` is called, it returns `HTTP 404 Not Found` with `ProblemDetails`.

## Documentation

For a comprehensive guide, detailed API reference, and advanced usage patterns, please visit the [Wiki](https://github.com/ulfbou/Zentient.Results.AspNetCore/wiki). ## Contributing

We welcome contributions! Please see our [Contributing Guide](https://github.com/ulfbou/Zentient.Results.AspNetCore/blob/main/CONTRIBUTING.md) for more details. ## License

Zentient.Results.AspNetCore is released under the [MIT License](https://github.com/ulfbou/Zentient.Results.AspNetCore/blob/main/LICENSE). ---
*Part of the Zentient .NET Libraries.*
