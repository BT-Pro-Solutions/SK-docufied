# Explanation
This code defines a C# plugin class, `TimePlugin`, that provides the current UTC time. It is structured to be used with Microsoft's Semantic Kernel framework by utilizing the `[KernelFunction]` attribute. The `GetCurrentUtcTime` method returns the current UTC time in the RFC1123 string format ("R"), making it accessible as a plugin function that can be called within the Semantic Kernel runtime. The class resides in the `StepwisePlannerMigration.Plugins` namespace.

```csharp
// Copyright (c) Microsoft. All rights reserved.

#pragma warning disable IDE0005 // Using directive is unnecessary

using System;
using System.ComponentModel;
using Microsoft.SemanticKernel;

#pragma warning restore IDE0005 // Using directive is unnecessary

namespace StepwisePlannerMigration.Plugins;

/// <summary>
/// Sample plugin which provides time information.
/// </summary>
public sealed class TimePlugin
{
    [KernelFunction]
    [Description("Retrieves the current time in UTC")]
    public string GetCurrentUtcTime() => DateTime.UtcNow.ToString("R");
}
```