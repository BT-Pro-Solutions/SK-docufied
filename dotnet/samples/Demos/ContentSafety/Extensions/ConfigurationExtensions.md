# Explanation

This file defines a static class `ConfigurationExtensions` in the `ContentSafety.Extensions` namespace, specifically designed to extend functionality for application configuration management in .NET projects. The core feature is the `GetValid<TOptions>` extension method for `IConfigurationRoot`, which retrieves a configuration section, binds it to a strongly typed object (`TOptions`), and validates the resulting options object using data annotations. If validation fails, it throws a `ValidationException`. This helps ensure that application configuration conforms to expected validation rules, improving runtime robustness and minimizing misconfiguration errors.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ContentSafety.Extensions;

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