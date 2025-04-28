# Explanation
The `StaticTextPlugin.cs` file defines a plugin class for use with Microsoft Semantic Kernel. The `StaticTextPlugin` class includes two static methods, both exposed as kernel functions with descriptive metadata:
- `Uppercase`: Takes an input string and returns it fully uppercased.
- `AppendDay`: Appends a given day value to a provided input string.

These methods are decorated with attributes to enable automatic discovery and description by Semantic Kernel, allowing integration into applications needing text transformation or augmentation capabilities.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace Plugins;

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
```