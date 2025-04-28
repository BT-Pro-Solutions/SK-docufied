# Explanation

This C# code demonstrates how to use the `InMemoryVectorStore` from Microsoft.SemanticKernel to create, serialize (save to disk), deserialize (load from disk), and perform vector-based searches on a collection of text records. It features two main usage examples:

1. **LoadStringListAndSearchAsync**: 
   - Loads a list of strings (from a resource file), embeds them using OpenAI's embedding models, creates a vector collection, and saves it to disk. If the collection file exists, it loads and searches it using a newly generated embedding for a search string.

2. **LoadTextSearchResultsAndSearchAsync**: 
   - Loads text search result entries (from a JSON resource), embeds them, creates a collection, and performs a vector search.

A sample data model (`DataModel`) is provided, annotated with attributes for vector store compatibility. The code uses dependency injection (e.g., for test output) and can be run as part of automated tests.

---

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json;
using Microsoft.Extensions.VectorData;
using Microsoft.SemanticKernel.Connectors.InMemory;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.Data;
using Microsoft.SemanticKernel.Embeddings;
using Resources;

namespace Memory;

/// <summary>
/// Sample showing how to create an <see cref="InMemoryVectorStore"/> collection from a list of strings
/// and then save it to disk so that it can be reloaded later.
/// </summary>
public class InMemoryVectorStore_LoadData(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task LoadStringListAndSearchAsync()
    {
        // Create a logging handler to output HTTP requests and responses
        var handler = new LoggingHandler(new HttpClientHandler(), this.Output);
        var httpClient = new HttpClient(handler);

        // Create an embedding generation service.
        var embeddingGenerationService = new OpenAITextEmbeddingGenerationService(
                modelId: TestConfiguration.OpenAI.EmbeddingModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey,
                httpClient: httpClient);

        // Construct an InMemory vector store.
        var vectorStore = new InMemoryVectorStore();
        var collectionName = "records";

        // Path to the file where the record collection will be saved to and loaded from.
        string filePath = Path.Combine(Path.GetTempPath(), "semantic-kernel-info.json");
        if (!File.Exists(filePath))
        {
            // Read a list of text strings from a file, to load into a new record collection.
            var skInfo = EmbeddedResource.Read("semantic-kernel-info.txt");
            var lines = skInfo!.Split('\n');

            // Delegate which will create a record.
            static DataModel CreateRecord(string text, ReadOnlyMemory<float> embedding)
            {
                return new()
                {
                    Key = Guid.NewGuid(),
                    Text = text,
                    Embedding = embedding
                };
            }

            // Create a record collection from a list of strings using the provided delegate.
            var collection = await vectorStore.CreateCollectionFromListAsync<Guid, DataModel>(
                collectionName, lines, embeddingGenerationService, CreateRecord);

            // Save the record collection to a file stream.
            using (FileStream fileStream = new(filePath, FileMode.OpenOrCreate))
            {
                await vectorStore.SerializeCollectionAsJsonAsync<Guid, DataModel>(collectionName, fileStream);
            }
        }

        // Load the record collection from the file stream and perform a search.
        using (FileStream fileStream = new(filePath, FileMode.Open))
        {
            var vectorSearch = await vectorStore.DeserializeCollectionFromJsonAsync<Guid, DataModel>(fileStream);

            // Search the collection using a vector search.
            var searchString = "What is the Semantic Kernel?";
            var searchVector = await embeddingGenerationService.GenerateEmbeddingAsync(searchString);
            var searchResult = await vectorSearch!.VectorizedSearchAsync(searchVector, new() { Top = 1 });
            var resultRecords = await searchResult.Results.ToListAsync();

            Console.WriteLine("Search string: " + searchString);
            Console.WriteLine("Result: " + resultRecords.First().Record.Text);
            Console.WriteLine();
        }
    }

    [Fact]
    public async Task LoadTextSearchResultsAndSearchAsync()
    {
        // Create an embedding generation service.
        var embeddingGenerationService = new OpenAITextEmbeddingGenerationService(
                modelId: TestConfiguration.OpenAI.EmbeddingModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey);

        // Construct an InMemory vector store.
        var vectorStore = new InMemoryVectorStore();
        var collectionName = "records";

        // Read a list of text strings from a file, to load into a new record collection.
        var searchResultsJson = EmbeddedResource.Read("what-is-semantic-kernel.json");
        var searchResults = JsonSerializer.Deserialize<List<TextSearchResult>>(searchResultsJson!);

        // Delegate which will create a record.
        static DataModel CreateRecord(TextSearchResult searchResult, ReadOnlyMemory<float> embedding)
        {
            return new()
            {
                Key = Guid.NewGuid(),
                Title = searchResult.Name,
                Text = searchResult.Value ?? string.Empty,
                Link = searchResult.Link,
                Embedding = embedding
            };
        }

        // Create a record collection from a list of strings using the provided delegate.
        var vectorSearch = await vectorStore.CreateCollectionFromTextSearchResultsAsync<Guid, DataModel>(
            collectionName, searchResults!, embeddingGenerationService, CreateRecord);

        // Search the collection using a vector search.
        var searchString = "What is the Semantic Kernel?";
        var searchVector = await embeddingGenerationService.GenerateEmbeddingAsync(searchString);
        var searchResult = await vectorSearch!.VectorizedSearchAsync(searchVector, new() { Top = 1 });
        var resultRecords = await searchResult.Results.ToListAsync();

        Console.WriteLine("Search string: " + searchString);
        Console.WriteLine("Result: " + resultRecords.First().Record.Text);
        Console.WriteLine();
    }

    /// <summary>
    /// Sample model class that represents a record entry.
    /// </summary>
    /// <remarks>
    /// Note that each property is decorated with an attribute that specifies how the property should be treated by the vector store.
    /// This allows us to create a collection in the vector store and upsert and retrieve instances of this class without any further configuration.
    /// </remarks>
    private sealed class DataModel
    {
        [VectorStoreRecordKey]
        public Guid Key { get; init; }

        [VectorStoreRecordData]
        public string? Title { get; init; }

        [VectorStoreRecordData]
        public string Text { get; init; }

        [VectorStoreRecordData]
        public string? Link { get; init; }

        [VectorStoreRecordVector(1536)]
        public ReadOnlyMemory<float> Embedding { get; init; }
    }
}
```
