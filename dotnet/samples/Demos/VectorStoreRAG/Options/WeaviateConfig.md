# Explanation
This C# code defines a configuration class, `WeaviateConfig`, intended to hold settings for connecting to a Weaviate service within the `VectorStoreRAG.Options` namespace. The class contains a constant specifying the configuration section name ("Weaviate") and a required string property (`Endpoint`) for storing the Weaviate service endpoint. The use of `[Required]` ensures that the `Endpoint` must be provided in the application's configuration.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Weaviate service settings.
/// </summary>
internal sealed class WeaviateConfig
{
    public const string ConfigSectionName = "Weaviate";

    [Required]
    public string Endpoint { get; set; } = string.Empty;
}
```