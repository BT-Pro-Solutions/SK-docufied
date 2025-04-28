# Explanation
This C# class defines configuration options for Azure OpenAI services within a home automation context. The `AzureOpenAIOptions` class contains properties for the chat model deployment name, endpoint URL, and API key, all of which are required. It uses data annotations to enforce that these properties must be set. The class also specifies a constant section name used for binding the configuration.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace HomeAutomation.Options;

/// <summary>
/// Azure OpenAI settings.
/// </summary>
public sealed class AzureOpenAIOptions
{
    public const string SectionName = "AzureOpenAI";

    [Required]
    public string ChatDeploymentName { get; set; } = string.Empty;

    [Required]
    public string Endpoint { get; set; } = string.Empty;

    [Required]
    public string ApiKey { get; set; } = string.Empty;
}
```