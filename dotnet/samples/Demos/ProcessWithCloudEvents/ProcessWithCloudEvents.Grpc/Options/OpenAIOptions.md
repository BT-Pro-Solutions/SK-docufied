# Explanation
This C# class defines configuration options for an OpenAI chat completion service. It includes properties for the chat model identifier and the API key, both of which are required. The class is intended to be used for binding configuration values from a configuration source, typically from a section named "OpenAI".

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ProcessWithCloudEvents.Grpc.Options;

/// <summary>
/// Configuration for OpenAI chat completion service.
/// </summary>
public class OpenAIOptions
{
    public const string SectionName = "OpenAI";

    [Required]
    public string ChatModelId { get; set; } = string.Empty;

    [Required]
    public string ApiKey { get; set; } = string.Empty;
}
```