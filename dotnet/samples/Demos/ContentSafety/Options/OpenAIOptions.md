# Explanation
This C# code defines an options class used to configure integration with the OpenAI chat completion service for an application. The `OpenAIOptions` class holds settings required for connecting to OpenAI, specifically the chat model ID and API key. Both properties are marked with the `[Required]` attribute to enforce their presence during configuration binding. The class also includes a constant `SectionName` used to group these settings logically in configuration files. This pattern is commonly used in ASP.NET Core applications for structured configuration management.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ContentSafety.Options;

/// <summary>
/// Configuration for OpenAI chat completion service.
/// </summary>
public class OpenAIOptions
{
    public const string SectionName = "OpenAI";

    [Required]
    public string ChatModelId { get; set; }

    [Required]
    public string ApiKey { get; set; }
}
```