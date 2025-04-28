# Explanation
This C# code defines an extension class, `ConfigurationExtensions`, providing a utility method for configuration validation in .NET applications. Specifically, the `GetValid<TOptions>` method extends `IConfigurationRoot` to retrieve a typed configuration section and validate it using data annotation attributes. If the configuration object is invalid, it throws a `ValidationException`. This approach helps ensure that application configuration settings are both present and valid at startup, improving reliability and error reporting.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;
using Microsoft.Extensions.Configuration;

namespace StepwisePlannerMigration.Extensions;

/// <summary>
/// Class with extension methods for app configuration.
/// </summary>
public static class ConfigurationExtensions
{
    /// <summary>
    /// Returns <typeparamref name="TOptions"/> if it's valid or throws <see cref="ValidationException"/>.
    /// </summary>
    public static TOptions GetValid<TOptions>(this IConfigurationRoot configurationRoot, string sectionName)
    {
        var options = configurationRoot.GetSection(sectionName).Get<TOptions>()!;

        Validator.ValidateObject(options, new(options));

        return options;
    }
}
```