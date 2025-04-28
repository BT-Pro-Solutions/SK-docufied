# Explanation

This C# code demonstrates how to use the Semantic Kernel SDK to interact with MistralAI's chat completion capabilities. It contains a test class, `MistralAI_ChatPrompt`, which showcases three main scenarios:
1. **GetChatMessageContentsAsync**: Sends a simple chat history to the MistralAI model and prints each response message. The prompt instructs the model to answer in French.
2. **GetStreamingChatMessageContentsAsync**: Demonstrates how to use MistralAI's streaming API to receive the response incrementally, printing each token/word as it is streamed.
3. **ChatPromptAsync**: Shows how to create a chat prompt using Semantic Kernel's function from a formatted prompt template, and invokes the kernel to get a complete response.

All examples use a test configuration for model and API key, and use .NET's async/await pattern for asynchronous API calls. The code is structured as xUnit tests.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.MistralAI;

namespace ChatCompletion;

/// <summary>
/// Demonstrates the use of chat prompts with MistralAI.
/// </summary>
public sealed class MistralAI_ChatPrompt(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task GetChatMessageContentsAsync()
    {
        var service = new MistralAIChatCompletionService(
            TestConfiguration.MistralAI.ChatModelId!,
            TestConfiguration.MistralAI.ApiKey!
        );

        var chatHistory = new ChatHistory
        {
            new ChatMessageContent(AuthorRole.System, "Respond in French."),
            new ChatMessageContent(AuthorRole.User, "What is the best French cheese?")
        };
        var response = await service.GetChatMessageContentsAsync(
            chatHistory, new MistralAIPromptExecutionSettings { MaxTokens = 500 });

        foreach (var message in response)
        {
            Console.WriteLine(message.Content);
        }
    }

    [Fact]
    public async Task GetStreamingChatMessageContentsAsync()
    {
        var service = new MistralAIChatCompletionService(
            TestConfiguration.MistralAI.ChatModelId!,
            TestConfiguration.MistralAI.ApiKey!
        );

        var chatHistory = new ChatHistory
        {
            new ChatMessageContent(AuthorRole.System, "Respond in French."),
            new ChatMessageContent(AuthorRole.User, "What is the best French cheese?")
        };
        var streamingChat = service.GetStreamingChatMessageContentsAsync(
            chatHistory, new MistralAIPromptExecutionSettings { MaxTokens = 500 });

        await foreach (var update in streamingChat)
        {
            Console.Write(update);
        }
    }

    [Fact]
    public async Task ChatPromptAsync()
    {
        const string ChatPrompt = """
            <message role="system">Respond in French.</message>
            <message role="user">What is the best French cheese?</message>
        """;

        var kernel = Kernel.CreateBuilder()
            .AddMistralChatCompletion(
                modelId: TestConfiguration.MistralAI.ChatModelId,
                apiKey: TestConfiguration.MistralAI.ApiKey)
            .Build();

        var chatSemanticFunction = kernel.CreateFunctionFromPrompt(
            ChatPrompt, new MistralAIPromptExecutionSettings { MaxTokens = 500 });
        var chatPromptResult = await kernel.InvokeAsync(chatSemanticFunction);

        Console.WriteLine(chatPromptResult);
    }
}
```