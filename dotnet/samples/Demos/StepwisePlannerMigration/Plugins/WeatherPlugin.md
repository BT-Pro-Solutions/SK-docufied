# Explanation

This code defines a sample plugin class, `WeatherPlugin`, which is intended for use with Microsoft's Semantic Kernel. The plugin provides a single method, `GetWeatherForCity`, that returns hardcoded, fake weather information for specific cities. It utilizes the `[KernelFunction]` attribute to indicate that the method can be used as a function within the kernel's context, and the `[Description]` attribute gives a brief description of the method for documentation or interface purposes. If a city name is not recognized, it returns a default weather value. This code is useful as a demonstration or testing utility when integrating plugins with the Semantic Kernel framework.

```csharp
// Copyright (c) Microsoft. All rights reserved.

#pragma warning disable IDE0005 // Using directive is unnecessary

using System.ComponentModel;
using Microsoft.SemanticKernel;

#pragma warning restore IDE0005 // Using directive is unnecessary

namespace StepwisePlannerMigration.Plugins;

/// <summary>
/// Sample plugin which provides fake weather information.
/// </summary>
public sealed class WeatherPlugin
{
    [KernelFunction]
    [Description("Gets the current weather for the specified city")]
    public string GetWeatherForCity(string cityName) =>
        cityName switch
        {
            "Boston" => "61 and rainy",
            "London" => "55 and cloudy",
            "Miami" => "80 and sunny",
            "Paris" => "60 and rainy",
            "Tokyo" => "50 and sunny",
            "Sydney" => "75 and sunny",
            "Tel Aviv" => "80 and sunny",
            _ => "31 and snowing",
        };
}
```