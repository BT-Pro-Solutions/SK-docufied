# Explanation
This C# code defines a configuration class named QdrantConfig, which holds configuration settings for integrating with a Qdrant vector database service. The class is intended for use in an application that interacts with Qdrant, possibly for AI or retrieval-augmented generation scenarios. Key configuration options include the Qdrant service host, port, whether to use HTTPS, and an optional API key for authentication. The Host property is marked as required to ensure the configuration cannot be considered valid unless it is provided.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Qdrant service settings.
/// </summary>
internal sealed class QdrantConfig
{
    public const string ConfigSectionName = "Qdrant";

    [Required]
    public string Host { get; set; } = string.Empty;

    public int Port { get; set; } = 6334;

    public bool Https { get; set; } = false;

    public string ApiKey { get; set; } = string.Empty;
}
```