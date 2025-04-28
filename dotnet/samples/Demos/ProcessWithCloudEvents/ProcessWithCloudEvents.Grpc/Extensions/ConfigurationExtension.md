# Explanation
This code defines an extension method for `IConfigurationRoot` that allows retrieving a configuration section as a strongly typed object with validation. The method `GetValid<TOptions>` reads a configuration section, binds it to an instance of the specified type `TOptions`, and then validates the object using data annotations. If the object is invalid, a `ValidationException` is thrown. This is useful for ensuring configuration data is correct and complete when the application starts.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ProcessWithCloudEvents.Grpc.Extensions;

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