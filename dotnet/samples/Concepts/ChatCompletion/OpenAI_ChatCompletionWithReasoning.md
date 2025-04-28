# Explanation

This C# code demonstrates how to use the Microsoft Semantic Kernel SDK with the OpenAI API to perform chat completions with reasoning capabilities. Specifically, it shows two ways to interact with OpenAI's chat models, including models capable of different "reasoning effort" levels:
1. Using the Kernel class with chat prompt syntax to invoke an AI-driven chat with specified reasoning effort.
2. Using the IChatCompletionService directly with a ChatHistory to submit user and developer messages programmatically.

The code uses Xunit tests to organize these two sample methods and assumes the presence of valid OpenAI API credentials. The ReasoningEffort parameter is used, which is supported by certain OpenAI models. The purpose is to illustrate how to integrate chat-based AI interaction within a .NET application using Semantic Kernel for advanced prompting and reasoning features.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using OpenAI.Chat;

namespace ChatCompletion;

// The following example shows how to use Semantic Kernel with OpenAI API
public class OpenAI_ChatCompletionWithReasoning(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Sample showing how to use <see cref="Kernel"/> with chat completion and chat prompt syntax.
    /// </summary>
    [Fact]
    public async Task ChatPromptWithReasoningAsync()
    {
        Console.WriteLine("======== Open AI - Chat Completion with Reasoning ========");

        Assert.NotNull(TestConfiguration.OpenAI.ChatModelId);
        Assert.NotNull(TestConfiguration.OpenAI.ApiKey);

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey)
            .Build();

        // Create execution settings with low reasoning effort.
        var executionSettings = new OpenAIPromptExecutionSettings //OpenAIPromptExecutionSettings
        {
            MaxTokens = 2000,
            ReasoningEffort = ChatReasoningEffortLevel.Low // Only available for reasoning models (i.e: o3-mini, o1, ...)
        };

        // Create KernelArguments using the execution settings.
        var kernelArgs = new KernelArguments(executionSettings);

        StringBuilder chatPrompt = new("""
                                   <message role="developer">You are an expert software engineer, specialized in the Semantic Kernel SDK and NET framework</message>
                                   <message role="user">Hi, Please craft me an example code in .NET using Semantic Kernel that implements a chat loop .</message>
                                   """);

        // Invoke the prompt with high reasoning effort.
        var reply = await kernel.InvokePromptAsync(chatPrompt.ToString(), kernelArgs);

        Console.WriteLine(reply);
    }

    /// <summary>
    /// Sample showing how to use <see cref="IChatCompletionService"/> directly with a <see cref="ChatHistory"/>.
    /// </summary>
    [Fact]
    public async Task ServicePromptWithReasoningAsync()
    {
        Assert.NotNull(TestConfiguration.OpenAI.ChatModelId);
        Assert.NotNull(TestConfiguration.OpenAI.ApiKey);

        Console.WriteLine("======== Open AI - Chat Completion with Reasoning ========");

        OpenAIChatCompletionService chatCompletionService = new(TestConfiguration.OpenAI.ChatModelId, TestConfiguration.OpenAI.ApiKey);

        // Create execution settings with low reasoning effort.
        var executionSettings = new OpenAIPromptExecutionSettings
        {
            MaxTokens = 2000,
            ReasoningEffort = ChatReasoningEffortLevel.Low // Only available for reasoning models (i.e: o3-mini, o1, ...)
        };

        // Create a ChatHistory and add messages.
        var chatHistory = new ChatHistory();
        chatHistory.AddDeveloperMessage(
            "You are an expert software engineer, specialized in the Semantic Kernel SDK and .NET framework.");
        chatHistory.AddUserMessage(
            "Hi, Please craft me an example code in .NET using Semantic Kernel that implements a chat loop.");

        // Instead of a prompt string, call GetChatMessageContentAsync with the chat history.
        var reply = await chatCompletionService.GetChatMessageContentAsync(
            chatHistory: chatHistory,
            executionSettings: executionSettings);

        Console.WriteLine(reply);
    }
}
```