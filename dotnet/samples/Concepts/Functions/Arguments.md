# Explanation

This C# code demonstrates how to use kernel arguments when invoking functions with the Microsoft.SemanticKernel library. The Arguments class contains an asynchronous test case (`RunAsync`) that creates a plugin with simple string manipulation functions and showcases different ways to invoke these functions using the Semantic Kernel's `KernelArguments`. It illustrates calling a function generically to directly get a value, as well as using the `FunctionResult` object to access both the value and its metadata. The example includes a static plugin with two kernel functions: one for converting a string to uppercase and another for appending the current day of the week to a given string.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using System.Globalization;
using Microsoft.SemanticKernel;

namespace Functions;

// This example shows how to use kernel arguments when invoking functions.
public class Arguments(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task RunAsync()
    {
        Console.WriteLine("======== Arguments ========");

        Kernel kernel = new();
        var textPlugin = kernel.ImportPluginFromType<StaticTextPlugin>();

        var arguments = new KernelArguments()
        {
            ["input"] = "Today is: ",
            ["day"] = DateTimeOffset.Now.ToString("dddd", CultureInfo.CurrentCulture)
        };

        // ** Different ways of executing functions with arguments **

        // Specify and get the value type as generic parameter
        string? resultValue = await kernel.InvokeAsync<string>(textPlugin["AppendDay"], arguments);
        Console.WriteLine($"string -> {resultValue}");

        // If you need to access the result metadata, you can use the non-generic version to get the FunctionResult
        FunctionResult functionResult = await kernel.InvokeAsync(textPlugin["AppendDay"], arguments);
        var metadata = functionResult.Metadata;

        // Specify the type from the FunctionResult
        Console.WriteLine($"FunctionResult.GetValue<string>() -> {functionResult.GetValue<string>()}");

        // FunctionResult.ToString() automatically converts the result to string
        Console.WriteLine($"FunctionResult.ToString() -> {functionResult}");
    }

    public sealed class StaticTextPlugin
    {
        [KernelFunction, Description("Change all string chars to uppercase")]
        public static string Uppercase([Description("Text to uppercase")] string input) =>
            input.ToUpperInvariant();

        [KernelFunction, Description("Append the day variable")]
        public static string AppendDay(
            [Description("Text to append to")] string input,
            [Description("Value of the day to append")] string day) =>
            input + day;
    }
}
```