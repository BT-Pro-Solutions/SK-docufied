# Explanation
This code defines an internal C# class, `Glossary`, which serves as a sample model for a glossary entry to be used with Microsoft's Vector Store integration. The class properties are annotated with specific attributes from `Microsoft.Extensions.VectorData`, which dictate how the properties should be mapped for storage and retrieval within a vector database. Notably:

- `Key` is marked as the record key.
- `Category` is set as a filterable field.
- `Term` and `Definition` are simple data fields.
- `DefinitionEmbedding` is a vector field with a fixed dimensionality of 1536, intended for storing vector representations of the definition (e.g., for semantic search).

This attribute-based configuration allows seamless integration with the vector store, enabling collection creation and CRUD operations without further setup.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.VectorData;

namespace GettingStartedWithVectorStores;

/// <summary>
/// Sample model class that represents a glossary entry.
/// </summary>
/// <remarks>
/// Note that each property is decorated with an attribute that specifies how the property should be treated by the vector store.
/// This allows us to create a collection in the vector store and upsert and retrieve instances of this class without any further configuration.
/// </remarks>
internal sealed class Glossary
{
    [VectorStoreRecordKey]
    public string Key { get; set; }

    [VectorStoreRecordData(IsFilterable = true)]
    public string Category { get; set; }

    [VectorStoreRecordData]
    public string Term { get; set; }

    [VectorStoreRecordData]
    public string Definition { get; set; }

    [VectorStoreRecordVector(Dimensions: 1536)]
    public ReadOnlyMemory<float> DefinitionEmbedding { get; set; }
}
```