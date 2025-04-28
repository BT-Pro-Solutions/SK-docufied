# Explanation

This code demonstrates how to use the Microsoft Semantic Kernel to interact with a Qdrant vector database for vector search capabilities. It provides two example test methods for ingesting data into Qdrant and performing searches:

1. **ExampleWithDIAsync**: Shows how to configure and use dependency injection for setting up embedding generation services, the Qdrant vector store, and a common processing class. The kernel is built using these services, and data ingestion/search is performed with key generation suitable for Qdrant.
2. **ExampleWithoutDIAsync**: Illustrates a manual setup where the Qdrant vector store and embedding service are created directly, without dependency injection. Similarly, data ingestion and search are performed with auto-generated keys appropriate for Qdrant.

The sample assumes a local Docker instance is available, as the fixture will start a Qdrant container. The processor logic for ingesting and searching is encapsulated in a shared class (`VectorStore_VectorSearch_MultiStore_Common`), making the code reusable across different vector database backends by swapping the storage implementation and key generator.

The code is intended for integration or unit tests, leveraging xUnit and dependency injection patterns.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Azure.Identity;
using Memory.VectorStoreFixtures;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.AzureOpenAI;
using Microsoft.SemanticKernel.Connectors.Qdrant;
using Qdrant.Client;

namespace Memory;

/// <summary>
/// An example showing how to use common code, that can work with any vector database, with a Qdrant database.
/// The common code is in the <see cref="VectorStore_VectorSearch_MultiStore_Common"/> class.
/// The common code ingests data into the vector store and then searches over that data.
/// This example is part of a set of examples each showing a different vector database.
///
/// For other databases, see the following classes:
/// <para><see cref="VectorStore_VectorSearch_MultiStore_AzureAISearch"/></para>
/// <para><see cref="VectorStore_VectorSearch_MultiStore_Redis"/></para>
/// <para><see cref="VectorStore_VectorSearch_MultiStore_InMemory"/></para>
/// <para><see cref="VectorStore_VectorSearch_MultiStore_Postgres"/></para>
///
/// To run this sample, you need a local instance of Docker running, since the associated fixture will try and start a Qdrant container in the local docker instance.
/// </summary>
public class VectorStore_VectorSearch_MultiStore_Qdrant(ITestOutputHelper output, VectorStoreQdrantContainerFixture qdrantFixture) : BaseTest(output), IClassFixture<VectorStoreQdrantContainerFixture>
{
    [Fact]
    public async Task ExampleWithDIAsync()
    {
        // Use the kernel for DI purposes.
        var kernelBuilder = Kernel
            .CreateBuilder();

        // Register an embedding generation service with the DI container.
        kernelBuilder.AddAzureOpenAITextEmbeddingGeneration(
            deploymentName: TestConfiguration.AzureOpenAIEmbeddings.DeploymentName,
            endpoint: TestConfiguration.AzureOpenAIEmbeddings.Endpoint,
            credential: new AzureCliCredential());

        // Initialize the Qdrant docker container via the fixtures and register the Qdrant VectorStore.
        await qdrantFixture.ManualInitializeAsync();
        kernelBuilder.AddQdrantVectorStore("localhost");

        // Register the test output helper common processor with the DI container.
        kernelBuilder.Services.AddSingleton<ITestOutputHelper>(this.Output);
        kernelBuilder.Services.AddTransient<VectorStore_VectorSearch_MultiStore_Common>();

        // Build the kernel.
        var kernel = kernelBuilder.Build();

        // Build a common processor object using the DI container.
        var processor = kernel.GetRequiredService<VectorStore_VectorSearch_MultiStore_Common>();

        // Run the process and pass a key generator function to it, to generate unique record keys.
        // The key generator function is required, since different vector stores may require different key types.
        // E.g. Qdrant supports Guid and ulong keys, but others may support strings only.
        await processor.IngestDataAndSearchAsync("skglossaryWithDI", () => Guid.NewGuid());
    }

    [Fact]
    public async Task ExampleWithoutDIAsync()
    {
        // Create an embedding generation service.
        var textEmbeddingGenerationService = new AzureOpenAITextEmbeddingGenerationService(
                TestConfiguration.AzureOpenAIEmbeddings.DeploymentName,
                TestConfiguration.AzureOpenAIEmbeddings.Endpoint,
                new AzureCliCredential());

        // Initialize the Qdrant docker container via the fixtures and construct the Qdrant VectorStore.
        await qdrantFixture.ManualInitializeAsync();
        var qdrantClient = new QdrantClient("localhost");
        var vectorStore = new QdrantVectorStore(qdrantClient);

        // Create the common processor that works for any vector store.
        var processor = new VectorStore_VectorSearch_MultiStore_Common(vectorStore, textEmbeddingGenerationService, this.Output);

        // Run the process and pass a key generator function to it, to generate unique record keys.
        // The key generator function is required, since different vector stores may require different key types.
        // E.g. Qdrant supports Guid and ulong keys, but others may support strings only.
        await processor.IngestDataAndSearchAsync("skglossaryWithoutDI", () => Guid.NewGuid());
    }
}
```
