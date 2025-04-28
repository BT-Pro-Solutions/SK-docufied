# Explanation

This code file, `KernelFunctionSamples.cs`, demonstrates how to create and invoke kernel functions in Ahead-of-Time (AOT) applications using the Microsoft Semantic Kernel library. It provides an example of defining a kernel function from a lambda expression, specifying custom JSON serialization contexts for type safety and compatibility in AOT scenarios. Specifically, the function takes a `Location` as input and returns a `Weather` object if the city is "Boston", otherwise it throws a `NotImplementedException`. The example shows how to set up the kernel, serialize arguments and results, and invoke the function asynchronously, printing the resulting `Weather` details.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json;
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;
using SemanticKernel.AotCompatibility.JsonSerializerContexts;
using SemanticKernel.AotCompatibility.Plugins;

namespace SemanticKernel.AotCompatibility;

/// <summary>
/// This class contains samples of how to create and invoke kernel functions in AOT applications.
/// </summary>
internal static class KernelFunctionSamples
{
    /// <summary>
    /// Creates a kernel function from a lambda and invokes it.
    /// </summary>
    /// <remarks>
    /// Other overloads of KernelFunctionFactory.CreateFromMethod can also be used to create functions,
    /// as well as the Kernel.CreateFunctionFromMethod extension methods.
    /// </remarks>
    public static async Task CreateFunctionFromLambda(IConfigurationRoot _)
    {
        Kernel kernel = new();

        // Create JsonSerializerOptions with custom JsonSerializerContexts for the Location and Weather types that are used in the lambda below.
        // This is necessary for JsonSerializer to infer the type information for these types in AOT applications.  
        JsonSerializerOptions options = new();
        options.TypeInfoResolverChain.Add(WeatherJsonSerializerContext.Default);
        options.TypeInfoResolverChain.Add(LocationJsonSerializerContext.Default);

        // Create a kernel function.
        KernelFunction function = KernelFunctionFactory.CreateFromMethod(
            method: (Location location) => location.City == "Boston" ? new Weather { Temperature = 61, Condition = "rainy" } : throw new NotImplementedException(),
            jsonSerializerOptions: options);

        // Invoke the function
        KernelArguments arguments = new() { ["location"] = new Location("USA", "Boston") };

        FunctionResult functionResult = await function.InvokeAsync(kernel, arguments);

        // Display the result
        Weather weather = functionResult.GetValue<Weather>()!;
        Console.WriteLine($"Temperature: {weather.Temperature}, Condition: {weather.Condition}");
    }
}
```