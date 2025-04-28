# Explanation
This C# class defines a simple data model called `TextDataModel` for storing records in a vector store. Each record consists of a unique identifier (`Key`), the text content (`Text`), and an embedding vector (`Embedding`). Attributes are used to indicate the role of each property in the vector store: the key, the data, and the vector with a specified dimension of 1536.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.VectorData;

namespace MCPServer.Resources;

/// <summary>
/// A simple data model for a record in the vector store.
/// </summary>
public class TextDataModel
{
    /// <summary>
    /// Unique identifier for the record.
    /// </summary>
    [VectorStoreRecordKey]
    public required Guid Key { get; init; }

    /// <summary>
    /// The text content of the record.
    /// </summary>
    [VectorStoreRecordData]
    public required string Text { get; init; }

    /// <summary>
    /// The embedding for the record.
    /// </summary>
    [VectorStoreRecordVector(1536)]
    public required ReadOnlyMemory<float> Embedding { get; init; }
}
```