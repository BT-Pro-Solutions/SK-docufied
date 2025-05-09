# Explanation

This C# program demonstrates how to set up a local AI chatbot using ONNX models within the Semantic Kernel framework. It loads a chat completion model (such as Phi-3) and a text embedding model (such as BGE-MICRO-V2), indexes fact files from a "Facts" directory into an in-memory vector store, and adds a plugin to enable semantic search across these facts. Users can then interactively ask questions in the console, and the assistant uses both AI chat completion and evidence from the indexed facts (retrieved via the semantic search plugin) to provide answers. The code also ensures proper disposal of resources and supports configuration via user secrets for model paths and identifiers.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using System.IO;
using System.Linq;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.VectorData;
using Microsoft.ML.OnnxRuntimeGenAI;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.InMemory;
using Microsoft.SemanticKernel.Connectors.Onnx;
using Microsoft.SemanticKernel.Data;
using Microsoft.SemanticKernel.Embeddings;
using Microsoft.SemanticKernel.PromptTemplates.Handlebars;

Console.OutputEncoding = System.Text.Encoding.UTF8;

// Ensure you follow the preparation steps provided in the README.md
var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

// Path to the folder of your downloaded ONNX PHI-3 model
var chatModelPath = config["Onnx:ModelPath"]!;
var chatModelId = config["Onnx:ModelId"] ?? "phi-3";

// Path to the file of your downloaded ONNX BGE-MICRO-V2 model
var embeddingModelPath = config["Onnx:EmbeddingModelPath"]!;

// Path to the vocab file your ONNX BGE-MICRO-V2 model
var embeddingVocabPath = config["Onnx:EmbeddingVocabPath"]!;

// If using Onnx GenAI 0.5.0 or later, the OgaHandle class must be used to track
// resources used by the Onnx services, before using any of the Onnx services.
using var ogaHandle = new OgaHandle();

// Load the services
var builder = Kernel.CreateBuilder()
    .AddOnnxRuntimeGenAIChatCompletion(chatModelId, chatModelPath)
    .AddBertOnnxTextEmbeddingGeneration(embeddingModelPath, embeddingVocabPath);

// Build Kernel
var kernel = builder.Build();

// Get the instances of the services
using var chatService = kernel.GetRequiredService<IChatCompletionService>() as OnnxRuntimeGenAIChatCompletionService;
var embeddingService = kernel.GetRequiredService<ITextEmbeddingGenerationService>();

// Create a vector store and a collection to store information
var vectorStore = new InMemoryVectorStore();
var collection = vectorStore.GetCollection<string, InformationItem>("ExampleCollection");
await collection.CreateCollectionIfNotExistsAsync();

// Save some information to the memory
var collectionName = "ExampleCollection";
foreach (var factTextFile in Directory.GetFiles("Facts", "*.txt"))
{
    var factContent = File.ReadAllText(factTextFile);
    await collection.UpsertAsync(new()
    {
        Id = Guid.NewGuid().ToString(),
        Text = factContent,
        Embedding = await embeddingService.GenerateEmbeddingAsync(factContent)
    });
}

// Add a plugin to search the database with.
var vectorStoreTextSearch = new VectorStoreTextSearch<InformationItem>(collection, embeddingService);
kernel.Plugins.Add(vectorStoreTextSearch.CreateWithSearch("SearchPlugin"));

// Start the conversation
while (true)
{
    // Get user input
    Console.ForegroundColor = ConsoleColor.White;
    Console.Write("User > ");
    var question = Console.ReadLine()!;

    // Clean resources and exit the demo if the user input is null or empty
    if (question is null || string.IsNullOrWhiteSpace(question))
    {
        // To avoid any potential memory leak all disposable
        // services created by the kernel are disposed
        DisposeServices(kernel);
        return;
    }

    // Invoke the kernel with the user input
    var response = kernel.InvokePromptStreamingAsync(
        promptTemplate: @"Question: {{input}}
        Answer the question using the memory content:
        {{#with (SearchPlugin-Search input)}}
          {{#each this}}
            {{this}}
            -----------------
          {{/each}}
        {{/with}}",
        templateFormat: "handlebars",
        promptTemplateFactory: new HandlebarsPromptTemplateFactory(),
        arguments: new KernelArguments()
        {
            { "input", question },
            { "collection", collectionName }
        });

    Console.Write("\nAssistant > ");

    await foreach (var message in response)
    {
        Console.Write(message);
    }

    Console.WriteLine();
}

static void DisposeServices(Kernel kernel)
{
    foreach (var target in kernel
        .GetAllServices<IChatCompletionService>()
        .OfType<IDisposable>())
    {
        target.Dispose();
    }
}

/// <summary>
/// Information item to represent the embedding data stored in the memory
/// </summary>
internal sealed class InformationItem
{
    [VectorStoreRecordKey]
    [TextSearchResultName]
    public string Id { get; set; } = string.Empty;

    [VectorStoreRecordData]
    [TextSearchResultValue]
    public string Text { get; set; } = string.Empty;

    [VectorStoreRecordVector(Dimensions: 384)]
    public ReadOnlyMemory<float> Embedding { get; set; }
}
```