# Explanation
This code defines a custom exception handler for scenarios involving content safety in an ASP.NET Core web application. The ContentSafetyExceptionHandler class implements the IExceptionHandler interface and is designed to intercept exceptions related to text moderation or attack detection (specifically TextModerationException or AttackDetectionException). When such exceptions occur, it crafts a problem details response with status code 400 (Bad Request), including the exception message and any extra data associated with the exception, and returns this formatted error information as a JSON response to the client. If the exception does not match these types, it allows the default handling to proceed.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using ContentSafety.Exceptions;
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;

namespace ContentSafety.Handlers;

/// <summary>
/// Exception handler for content safety scenarios.
/// It allows to return formatted content back to the client with exception details.
/// </summary>
public class ContentSafetyExceptionHandler : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
    {
        if (exception is not TextModerationException and not AttackDetectionException)
        {
            return false;
        }

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status400BadRequest,
            Title = "Bad Request",
            Detail = exception.Message,
            Extensions = (IDictionary<string, object?>)exception.Data
        };

        httpContext.Response.StatusCode = StatusCodes.Status400BadRequest;

        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}
```