# Explanation

This C# code demonstrates how to use Microsoft's Semantic Kernel with the Ollama API for generating text embeddings. The class `Ollama_EmbeddingGeneration` encapsulates an xUnit test method that configures a Semantic Kernel instance with the Ollama embedding generation service. It then generates embeddings for a given piece of dialogue text and outputs the number of embeddings created. This example is useful for developers looking to integrate Ollama's embedding models with Semantic Kernel in a .NET test environment, particularly using dependency injection and asynchronous patterns.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Embeddings;
using xRetry;

#pragma warning disable format // Format item can be simplified
#pragma warning disable CA1861 // Avoid constant arrays as arguments

namespace Memory;

// The following example shows how to use Semantic Kernel with Ollama API.
public class Ollama_EmbeddingGeneration(ITestOutputHelper output) : BaseTest(output)
{
    [RetryFact(typeof(HttpOperationException))]
    public async Task RunEmbeddingAsync()
    {
        Assert.NotNull(TestConfiguration.Ollama.EmbeddingModelId);

        Console.WriteLine("\n======= Ollama - Embedding Example ========\n");

        Kernel kernel = Kernel.CreateBuilder()
            .AddOllamaTextEmbeddingGeneration(
                endpoint: new Uri(TestConfiguration.Ollama.Endpoint),
                modelId: TestConfiguration.Ollama.EmbeddingModelId)
            .Build();

        var embeddingGenerator = kernel.GetRequiredService<ITextEmbeddingGenerationService>();

        // Generate embeddings for each chunk.
        var embeddings = await embeddingGenerator.GenerateEmbeddingsAsync(["John: Hello, how are you?\nRoger: Hey, I'm Roger!"]);

        Console.WriteLine($"Generated {embeddings.Count} embeddings for the provided text");
    }
}
```