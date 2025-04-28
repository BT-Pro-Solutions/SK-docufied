# Explanation
This C# code defines a configuration class `OpenAIEmbeddingsConfig` for specifying settings related to OpenAI embeddings. It includes a constant for the configuration section name and a required property for the embeddings model name.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ChatWithAgent.Configuration;

/// <summary>
/// OpenAI embeddings configuration.
/// </summary>
public sealed class OpenAIEmbeddingsConfig
{
    /// <summary>
    /// Configuration section name.
    /// </summary>
    public const string ConfigSectionName = "OpenAIEmbeddings";

    /// <summary>
    /// The name of the embeddings model.
    /// </summary>
    [Required]
    public string ModelName { get; set; } = string.Empty;
}
```