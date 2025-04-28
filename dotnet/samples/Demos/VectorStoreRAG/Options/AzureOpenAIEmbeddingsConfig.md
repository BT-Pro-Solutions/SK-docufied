# Explanation
This code defines a configuration class for an Azure OpenAI Embeddings service in a .NET application. The class, AzureOpenAIEmbeddingsConfig, is marked as internal and is intended to be used within the VectorStoreRAG.Options namespace. It has two required string properties: DeploymentName and Endpoint, both of which must be provided for the configuration to be valid. The class also exposes the name of the configuration section expected in configuration files via the ConfigSectionName constant.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Azure OpenAI Embeddings service settings.
/// </summary>
internal sealed class AzureOpenAIEmbeddingsConfig
{
    public const string ConfigSectionName = "AzureOpenAIEmbeddings";

    [Required]
    public string DeploymentName { get; set; } = string.Empty;

    [Required]
    public string Endpoint { get; set; } = string.Empty;
}
```