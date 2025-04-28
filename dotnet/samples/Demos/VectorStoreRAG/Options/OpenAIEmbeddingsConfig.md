# Explanation
This C# file defines the configuration settings for an OpenAI Embeddings service within the VectorStoreRAG application. The OpenAIEmbeddingsConfig class includes properties for the model ID, API key, and (optionally) the organization ID—all of which are required except for OrgId, which is nullable. The class uses .NET's data annotations to enforce required fields. The ConfigSectionName constant specifies the configuration section name expected in the application's configuration files.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// OpenAI Embeddings service settings.
/// </summary>
internal sealed class OpenAIEmbeddingsConfig
{
    public const string ConfigSectionName = "OpenAIEmbeddings";

    [Required]
    public string ModelId { get; set; } = string.Empty;

    [Required]
    public string ApiKey { get; set; } = string.Empty;

    [Required]
    public string? OrgId { get; set; } = null;
}
```