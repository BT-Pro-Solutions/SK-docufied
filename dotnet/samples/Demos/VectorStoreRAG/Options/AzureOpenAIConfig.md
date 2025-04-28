# Explanation
The `AzureOpenAIConfig` class defines configuration settings for connecting to the Azure OpenAI service in a .NET application. It specifies properties for the chat model deployment name and the service endpoint, both of which are required and expected to be set via configuration (e.g., an appsettings file, environment variables). The class is intended to be used for binding configuration sections and enforcing that necessary fields are provided for Azure OpenAI access.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Azure OpenAI service settings.
/// </summary>
internal sealed class AzureOpenAIConfig
{
    public const string ConfigSectionName = "AzureOpenAI";

    [Required]
    public string ChatDeploymentName { get; set; } = string.Empty;

    [Required]
    public string Endpoint { get; set; } = string.Empty;
}
```