# Explanation

This code defines a static helper class `VectorStore_ConsumeFromMemoryStore_Common` used for populating a memory store with sample data in the context of Microsoft Semantic Kernel's vector store abstractions. 

The class provides methods to:
- Create a new collection in an `IMemoryStore`.
- Generate and insert multiple `MemoryRecord` instances containing sample textual data and their embeddings (vector representations) into the collection.

The sample records showcase how the Semantic Kernel's vector store connectors enable a "model first" approach for defining data schemas and mappings for interacting with vector databases. Each record demonstrates storing both raw text data and associated metadata, some of which reference external sources. Embeddings for these records are asynchronously generated via an `ITextEmbeddingGenerationService`.

Importantly, this code is intended as common functionality for a set of example applications that demonstrate consuming data previously uploaded via the `MemoryStore` abstractions, and it highlights how to bridge data between the older IMemoryStore abstraction and the new Vector Store abstraction.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel.Embeddings;
using Microsoft.SemanticKernel.Memory;

namespace Memory;

/// <summary>
/// Common code for all examples showing how to use the VectorStore abstractions to consume data that was
/// uploaded using the MemoryStore abstractions.
/// </summary>
/// <remarks>
/// The IMemoryStore abstraction has limitations that constrain its use in many scenarios
/// e.g. it only supports a single fixed schema and does not allow search filtering.
/// To provide more flexibility, the Vector Store abstraction has been introduced.
///
/// The run these examples, please see:
/// <see cref="VectorStore_ConsumeFromMemoryStore_AzureAISearch"/>
/// <see cref="VectorStore_ConsumeFromMemoryStore_Qdrant"/>
/// <see cref="VectorStore_ConsumeFromMemoryStore_Redis"/>
/// </remarks>
public static class VectorStore_ConsumeFromMemoryStore_Common
{
    public static async Task CreateCollectionAndAddSampleDataAsync(IMemoryStore memoryStore, string collectionName, ITextEmbeddingGenerationService textEmbeddingService)
    {
        // Build a collection with sample data using the MemoryStore abstraction.
        await memoryStore.CreateCollectionAsync(collectionName);
        await foreach (var memoryRecord in CreateSampleDataAsync(textEmbeddingService))
        {
            await memoryStore.UpsertAsync(collectionName, memoryRecord);
        }
    }

    private static async IAsyncEnumerable<MemoryRecord> CreateSampleDataAsync(ITextEmbeddingGenerationService textEmbeddingService)
    {
        var dateTimeOffset = new DateTimeOffset(2023, 10, 10, 0, 0, 0, TimeSpan.Zero);

        // Record 1.
        var text1 = """
            The Semantic Kernel Vector Store connectors use a model first approach to interacting with databases.
            This means that the first step is to define a data model that maps to the storage schema.
            To help the connectors create collections of records and map to the storage schema, the model can
            be annotated to indicate the function of each property.
            """;
        var embedding1 = await textEmbeddingService.GenerateEmbeddingAsync(text1);

        yield return new MemoryRecord(
            new MemoryRecordMetadata(
                isReference: false,
                id: "11111111-1111-1111-1111-111111111111",
                text: text1,
                description: "Describes the model first approach of Vector Store abstractions in SK.",
                externalSourceName: string.Empty,
                additionalMetadata: "sample: 1"),
            embedding1,
            key: "11111111-1111-1111-1111-111111111111",
            dateTimeOffset);

        // Record 2.
        var text2 = """The underlying implementation of what a collection is, will vary by connector and is influenced by how each database groups and indexes records.""";
        var embedding2 = await textEmbeddingService.GenerateEmbeddingAsync(text2);

        yield return new MemoryRecord(
            new MemoryRecordMetadata(
                isReference: true,
                id: "22222222-2222-2222-2222-222222222222",
                text: string.Empty,
                description: "Describes how collections are mapped in different stores.",
                externalSourceName: "https://learn.microsoft.com/en-us/semantic-kernel/concepts/vector-store-connectors/data-architecture#collections-in-different-databases",
                additionalMetadata: "sample: 2"),
            embedding2,
            key: "22222222-2222-2222-2222-222222222222",
            dateTimeOffset);

        // Record 3.
        var text3 = """
            The Semantic Kernel Vector Store connectors use a model first approach to interacting with databases.
            All methods to upsert or get records use strongly typed model classes. The properties on these classes are decorated with attributes that indicate the purpose of each property.
            """;
        var embedding3 = await textEmbeddingService.GenerateEmbeddingAsync(text3);

        yield return new MemoryRecord(
            new MemoryRecordMetadata(
                isReference: true,
                id: "33333333-3333-3333-3333-333333333333",
                text: string.Empty,
                description: "Describes the strong typing of Vector Store connectors.",
                externalSourceName: "https://learn.microsoft.com/en-us/semantic-kernel/concepts/vector-store-connectors/defining-your-data-model#overview",
                additionalMetadata: "sample: 3"),
            embedding3,
            key: "33333333-3333-3333-3333-333333333333",
            dateTimeOffset);
    }
}
```