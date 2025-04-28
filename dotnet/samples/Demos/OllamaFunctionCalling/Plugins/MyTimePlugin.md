# Explanation
This C# code defines a simple plugin class named MyTimePlugin within the OllamaFunctionCalling namespace. The class contains a single method, Time(), which returns the current system time as a DateTimeOffset. The method is decorated with the KernelFunction attribute and a Description attribute, indicating that it is intended to be used as a semantic kernel function (for example, in Microsoft's Semantic Kernel framework) for retrieving the current time.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace OllamaFunctionCalling;

/// <summary>
/// Simple plugin that just returns the time.
/// </summary>
public class MyTimePlugin
{
    [KernelFunction, Description("Get the current time")]
    public DateTimeOffset Time() => DateTimeOffset.Now;
}
```