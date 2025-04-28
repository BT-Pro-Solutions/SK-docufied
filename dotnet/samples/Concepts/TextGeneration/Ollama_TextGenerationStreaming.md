# Explanation

This C# code demonstrates how to interact with the Ollama Text Generation API using the Microsoft Semantic Kernel framework. It provides two test methods, each showing how to stream text generation results from Ollama:

1. `RunKernelStreamingExampleAsync`: Uses the `Kernel.InvokePromptStreamingAsync` method to stream responses to a prompt ("What is New York?") via the kernel abstraction.
2. `RunServiceStreamingExampleAsync`: Directly uses the injected `ITextGenerationService` to stream generated text content for the same prompt.

The code includes setup for the Ollama endpoint and model ID, both of which are fetched from test configurations. Output is printed to the console as it streams in, making it suitable for testing and demonstration purposes. The code assumes the existence of a testing framework (like xUnit) and related test infrastructure (e.g., `BaseTest`, `ITestOutputHelper`, and `TestConfiguration`).

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.TextGeneration;

#pragma warning disable format // Format item can be simplified
#pragma warning disable CA1861 // Avoid constant arrays as arguments

namespace TextGeneration;

// The following example shows how to use Semantic Kernel with Ollama Text Generation API.
public class Ollama_TextGenerationStreaming(ITestOutputHelper helper) : BaseTest(helper)
{
    [Fact]
    public async Task RunKernelStreamingExampleAsync()
    {
        Assert.NotNull(TestConfiguration.Ollama.ModelId);

        string model = TestConfiguration.Ollama.ModelId;

        Console.WriteLine($"\n======== Ollama {model} streaming example ========\n");

        Kernel kernel = Kernel.CreateBuilder()
            .AddOllamaTextGeneration(
                endpoint: new Uri(TestConfiguration.Ollama.Endpoint),
                modelId: model)
            .Build();

        await foreach (string text in kernel.InvokePromptStreamingAsync<string>("Question: {{$input}}; Answer:", new() { ["input"] = "What is New York?" }))
        {
            Console.Write(text);
        }
    }

    [Fact]
    public async Task RunServiceStreamingExampleAsync()
    {
        Assert.NotNull(TestConfiguration.Ollama.ModelId);

        string model = TestConfiguration.Ollama.ModelId;

        Console.WriteLine($"\n======== Ollama {model} streaming example ========\n");

        Kernel kernel = Kernel.CreateBuilder()
            .AddOllamaTextGeneration(
                endpoint: new Uri(TestConfiguration.Ollama.Endpoint),
                modelId: model)
            .Build();

        var service = kernel.GetRequiredService<ITextGenerationService>();

        await foreach (var content in service.GetStreamingTextContentsAsync("Question: What is New York?; Answer:"))
        {
            Console.Write(content);
        }
    }
}
```