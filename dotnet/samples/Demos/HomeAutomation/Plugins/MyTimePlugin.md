# Explanation
This C# code defines a simple plugin class `MyTimePlugin` within the `HomeAutomation.Plugins` namespace. The plugin includes a single method `Time` that returns the current date and time as a `DateTimeOffset` object. The method is annotated with `[KernelFunction]` to indicate it is a function usable by the Semantic Kernel framework, and a `Description` attribute provides metadata describing the function's purpose.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace HomeAutomation.Plugins;

/// <summary>
/// Simple plugin that just returns the time.
/// </summary>
public class MyTimePlugin
{
    [KernelFunction, Description("Get the current time")]
    public DateTimeOffset Time() => DateTimeOffset.Now;
}
```