# Explanation
This C# code demonstrates how to use Semantic Kernel with the ONNX GenAI API for text embedding generation. It provides two examples—one using the Semantic Kernel's `Kernel` builder and another using a direct `ServiceCollection` for dependency injection. Both examples show how to configure ONNX embedding model and vocab file paths, instantiate an embedding generation service, and generate text embeddings asynchronously. This functionality is useful for transforming text into vector representations, which is a foundational step in many natural language processing scenarios, such as semantic search or similarity ranking.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Embeddings;

namespace Memory;

// The following example shows how to use Semantic Kernel with Onnx GenAI API.
public class Onnx_EmbeddingGeneration(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Example using the service directly to get embeddings
    /// </summary>
    /// <remarks>
    /// Configuration example:
    /// <list type="table">
    /// <item>
    /// <term>EmbeddingModelPath:</term>
    /// <description>D:\huggingface\bge-micro-v2\onnx\model.onnx</description>
    /// </item>
    /// <item>
    /// <term>EmbeddingVocabPath:</term>
    /// <description>D:\huggingface\bge-micro-v2\vocab.txt</description>
    /// </item>
    /// </list>
    /// </remarks>
    [Fact]
    public async Task RunEmbeddingAsync()
    {
        Assert.NotNull(TestConfiguration.Onnx.EmbeddingModelPath); // dotnet user-secrets set "Onnx:EmbeddingModelPath" "<model-file-path>"
        Assert.NotNull(TestConfiguration.Onnx.EmbeddingVocabPath); // dotnet user-secrets set "Onnx:EmbeddingVocabPath" "<vocab-file-path>"

        Console.WriteLine("\n======= Onnx - Embedding Example ========\n");

        Kernel kernel = Kernel.CreateBuilder()
            .AddBertOnnxTextEmbeddingGeneration(TestConfiguration.Onnx.EmbeddingModelPath, TestConfiguration.Onnx.EmbeddingVocabPath)
            .Build();

        var embeddingGenerator = kernel.GetRequiredService<ITextEmbeddingGenerationService>();

        // Generate embeddings for each chunk.
        var embeddings = await embeddingGenerator.GenerateEmbeddingsAsync(["John: Hello, how are you?\nRoger: Hey, I'm Roger!"]);

        Console.WriteLine($"Generated {embeddings.Count} embeddings for the provided text");
    }

    /// <summary>
    /// Example using the service collection directly to get embeddings
    /// </summary>
    /// <remarks>
    /// Configuration example:
    /// <list type="table">
    /// <item>
    /// <term>EmbeddingModelPath:</term>
    /// <description>D:\huggingface\bge-micro-v2\onnx\model.onnx</description>
    /// </item>
    /// <item>
    /// <term>EmbeddingVocabPath:</term>
    /// <description>D:\huggingface\bge-micro-v2\vocab.txt</description>
    /// </item>
    /// </list>
    /// </remarks>
    [Fact]
    public async Task RunServiceCollectionEmbeddingAsync()
    {
        Assert.NotNull(TestConfiguration.Onnx.EmbeddingModelPath); // dotnet user-secrets set "Onnx:EmbeddingModelPath" "<model-file-path>"
        Assert.NotNull(TestConfiguration.Onnx.EmbeddingVocabPath); // dotnet user-secrets set "Onnx:EmbeddingVocabPath" "<vocab-file-path>"

        Console.WriteLine("\n======= Onnx - Embedding Example ========\n");

        var services = new ServiceCollection()
            .AddBertOnnxTextEmbeddingGeneration(TestConfiguration.Onnx.EmbeddingModelPath, TestConfiguration.Onnx.EmbeddingVocabPath);
        var provider = services.BuildServiceProvider();
        var embeddingGenerator = provider.GetRequiredService<ITextEmbeddingGenerationService>();

        // Generate embeddings for each chunk.
        var embeddings = await embeddingGenerator.GenerateEmbeddingsAsync(["John: Hello, how are you?\nRoger: Hey, I'm Roger!"]);

        Console.WriteLine($"Generated {embeddings.Count} embeddings for the provided text");
    }
}
```