# Explanation

This code defines a configuration class RagConfig for an application focused on Retrieval-Augmented Generation (RAG) using vector stores. The class contains several properties, many marked as required, that enable customization of AI service selection, document collection setup, batch processing parameters, and PDF input files. The configuration is likely intended to be loaded from a configuration file (such as appsettings.json) under the "Rag" section, allowing runtime control over core AI and data-loading aspects of the application.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Contains settings to control the RAG experience.
/// </summary>
internal sealed class RagConfig
{
    public const string ConfigSectionName = "Rag";

    [Required]
    public string AIChatService { get; set; } = string.Empty;

    [Required]
    public string AIEmbeddingService { get; set; } = string.Empty;

    [Required]
    public bool BuildCollection { get; set; } = true;

    [Required]
    public string CollectionName { get; set; } = string.Empty;

    [Required]
    public int DataLoadingBatchSize { get; set; } = 2;

    [Required]
    public int DataLoadingBetweenBatchDelayInMilliseconds { get; set; } = 0;

    [Required]
    public string[]? PdfFilePaths { get; set; }

    [Required]
    public string VectorStoreType { get; set; } = string.Empty;
}
```