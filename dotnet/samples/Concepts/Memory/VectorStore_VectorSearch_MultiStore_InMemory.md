# Explanation

This C# code demonstrates how to use an in-memory vector store for vector search within the Microsoft Semantic Kernel framework. It provides two examples: one using dependency injection (DI) to register services and build the vector search pipeline, and another without DI, where services are instantiated directly.

The in-memory vector store showcases a simple vector database implementation suitable for prototyping or local development, and serves as a reference for using the same common vector search code with other databases (such as Azure AI Search, Redis, Qdrant, and Postgres). The common ingestion and search logic is encapsulated in the `VectorStore_VectorSearch_MultiStore_Common` class and operates over a generic vector store interface.

Key points:
- The code uses Azure OpenAI for text embedding generation.
- Each example demonstrates how to ingest data and perform search using a customizable key generator (supporting multiple key types).
- Dependency injection is leveraged to easily swap implementations and manage lifetimes of services.
- The file contains references to related examples for other vector store backends.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Azure.Identity;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.AzureOpenAI;
using Microsoft.SemanticKernel.Connectors.InMemory;

namespace Memory;

/// <summary>
/// An example showing how to use common code, that can work with any vector database, with the InMemory vector store.
/// The common code is in the <see cref="VectorStore_VectorSearch_MultiStore_Common"/> class.
/// The common code ingests data into the vector store and then searches over that data.
/// This example is part of a set of examples each showing a different vector database.
///
/// For other databases, see the following classes:
/// <para><see cref="VectorStore_VectorSearch_MultiStore_AzureAISearch"/></para>
/// <para><see cref="VectorStore_VectorSearch_MultiStore_Redis"/></para>
/// <para><see cref="VectorStore_VectorSearch_MultiStore_Qdrant"/></para>
/// <para><see cref="VectorStore_VectorSearch_MultiStore_Postgres"/></para>
/// </summary>
public class VectorStore_VectorSearch_MultiStore_InMemory(ITestOutputHelper output) : BaseTest(output)
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

        // Register the InMemory VectorStore.
        kernelBuilder.AddInMemoryVectorStore();

        // Register the test output helper common processor with the DI container.
        kernelBuilder.Services.AddSingleton<ITestOutputHelper>(this.Output);
        kernelBuilder.Services.AddTransient<VectorStore_VectorSearch_MultiStore_Common>();

        // Build the kernel.
        var kernel = kernelBuilder.Build();

        // Build a common processor object using the DI container.
        var processor = kernel.GetRequiredService<VectorStore_VectorSearch_MultiStore_Common>();

        // Run the process and pass a key generator function to it, to generate unique record keys.
        // The key generator function is required, since different vector stores may require different key types.
        // E.g. InMemory supports any comparible type, but others may only support string or Guid or ulong, etc.
        // For this example we'll use int.
        var uniqueId = 0;
        await processor.IngestDataAndSearchAsync("skglossaryWithDI", () => uniqueId++);
    }

    [Fact]
    public async Task ExampleWithoutDIAsync()
    {
        // Create an embedding generation service.
        var textEmbeddingGenerationService = new AzureOpenAITextEmbeddingGenerationService(
                TestConfiguration.AzureOpenAIEmbeddings.DeploymentName,
                TestConfiguration.AzureOpenAIEmbeddings.Endpoint,
                new AzureCliCredential());

        // Construct the InMemory VectorStore.
        var vectorStore = new InMemoryVectorStore();

        // Create the common processor that works for any vector store.
        var processor = new VectorStore_VectorSearch_MultiStore_Common(vectorStore, textEmbeddingGenerationService, this.Output);

        // Run the process and pass a key generator function to it, to generate unique record keys.
        // The key generator function is required, since different vector stores may require different key types.
        // E.g. InMemory supports any comparible type, but others may only support string or Guid or ulong, etc.
        // For this example we'll use int.
        var uniqueId = 0;
        await processor.IngestDataAndSearchAsync("skglossaryWithoutDI", () => uniqueId++);
    }
}
```