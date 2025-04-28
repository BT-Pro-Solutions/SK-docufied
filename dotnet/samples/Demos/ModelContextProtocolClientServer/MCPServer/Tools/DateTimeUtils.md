# Explanation
This C# code defines a utility class `DateTimeUtils` within the `MCPServer.Tools` namespace. The class contains a single static method, `GetCurrentDateTimeInUtc()`, which returns the current UTC date and time as a formatted string (`"yyyy-MM-dd HH:mm:ss"`). The method is marked with attributes to integrate with the Microsoft Semantic Kernel framework, making it accessible as a kernel function with a descriptive annotation.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace MCPServer.Tools;

/// <summary>
/// A collection of utility methods for working with date time.
/// </summary>
internal sealed class DateTimeUtils
{
    /// <summary>
    /// Retrieves the current date time in UTC.
    /// </summary>
    /// <returns>The current date time in UTC.</returns>
    [KernelFunction, Description("Retrieves the current date time in UTC.")]
    public static string GetCurrentDateTimeInUtc()
    {
        return DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss");
    }
}
```