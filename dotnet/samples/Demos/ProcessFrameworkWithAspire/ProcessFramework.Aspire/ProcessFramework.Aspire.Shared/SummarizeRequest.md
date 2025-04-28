# Explanation
This C# class defines a simple data transfer object named `SummarizeRequest` within the `ProcessFramework.Aspire.Shared` namespace. It is intended to represent a request that contains text to be summarized. The class includes a single property, `TextToSummarize`, which holds the text that should be summarized. This property is initialized to an empty string by default.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace ProcessFramework.Aspire.Shared;

/// <summary>
/// Represents a request to summarize a given text.
/// </summary>
public class SummarizeRequest
{
    /// <summary>
    /// Gets or sets the text to be summarized.
    /// </summary>
    public string TextToSummarize { get; set; } = string.Empty;
}
```