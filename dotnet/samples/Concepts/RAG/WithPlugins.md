# Explanation

This C# code demonstrates how to augment Large Language Model (LLM) responses with Retrieval-Augmented Generation (RAG) using plugins within the Microsoft Semantic Kernel framework. There are three test cases shown:

1. **Custom Plugin with RAG**: Shows how to integrate a custom plugin (`CustomPlugin`) that simulates searching a vector database, allowing the LLM to access external data based on user queries.

2. **RAG with TextMemoryPlugin**: Demonstrates using Microsoft.SemanticKernel's built-in `TextMemoryPlugin` to retrieve contextual information from a local Chroma vector database (requires a running Chroma server).

3. **RAG with ChatGPT Retrieval Plugin**: Illustrates invoking the OpenAI ChatGPT Retrieval Plugin via its OpenAPI interface, with authentication, to fetch relevant documents from an external database.

These tests highlight how plugins expand LLM capabilities by providing external context retrieval, helping the model generate more accurate and relevant responses.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Net.Http.Headers;
using System.Text.Json;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.Chroma;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.Memory;
using Resources;

namespace RAG;

public class WithPlugins(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task RAGWithCustomPluginAsync()
    {
        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(TestConfiguration.OpenAI.ChatModelId, TestConfiguration.OpenAI.ApiKey)
            .Build();

        kernel.ImportPluginFromType<CustomPlugin>();

        var result = await kernel.InvokePromptAsync("{{search 'budget by year'}} What is my budget for 2024?");

        Console.WriteLine(result);
    }

    /// <summary>
    /// Shows how to use RAG pattern with <see cref="Microsoft.SemanticKernel.Plugins.Memory.TextMemoryPlugin"/>.
    /// </summary>
    [Fact(Skip = "Requires Chroma server up and running")]
    public async Task RAGWithTextMemoryPluginAsync()
    {
        var memory = new MemoryBuilder()
            .WithMemoryStore(new ChromaMemoryStore("http://localhost:8000"))
            .WithOpenAITextEmbeddingGeneration(TestConfiguration.OpenAI.EmbeddingModelId, TestConfiguration.OpenAI.ApiKey)
            .Build();

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(TestConfiguration.OpenAI.ChatModelId, TestConfiguration.OpenAI.ApiKey)
            .Build();

        kernel.ImportPluginFromObject(new Microsoft.SemanticKernel.Plugins.Memory.TextMemoryPlugin(memory));

        var result = await kernel.InvokePromptAsync("{{recall 'budget by year' collection='finances'}} What is my budget for 2024?");

        Console.WriteLine(result);
    }

    /// <summary>
    /// Shows how to use RAG pattern with ChatGPT Retrieval Plugin.
    /// </summary>
    [Fact(Skip = "Requires ChatGPT Retrieval Plugin and selected vector DB server up and running")]
    public async Task RAGWithChatGPTRetrievalPluginAsync()
    {
        var openApi = EmbeddedResource.ReadStream("chat-gpt-retrieval-plugin-open-api.yaml");

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(TestConfiguration.OpenAI.ChatModelId, TestConfiguration.OpenAI.ApiKey)
            .Build();

        await kernel.ImportPluginFromOpenApiAsync("ChatGPTRetrievalPlugin", openApi!, executionParameters: new(authCallback: async (request, cancellationToken) =>
        {
            request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", TestConfiguration.ChatGPTRetrievalPlugin.Token);
        }));

        const string Query = "What is my budget for 2024?";
        var function = KernelFunctionFactory.CreateFromPrompt("{{search queries=$queries}} {{$query}}");

        var arguments = new KernelArguments
        {
            ["query"] = Query,
            ["queries"] = JsonSerializer.Serialize(new List<object> { new { query = Query, top_k = 1 } }),
        };

        var result = await kernel.InvokeAsync(function, arguments);

        Console.WriteLine(result);
    }

    #region Custom Plugin

    private sealed class CustomPlugin
    {
        [KernelFunction]
        public async Task<string> SearchAsync(string query)
        {
            // Here will be a call to vector DB, return example result for demo purposes
            return "Year Budget 2020 100,000 2021 120,000 2022 150,000 2023 200,000 2024 364,000";
        }
    }

    #endregion
}
```