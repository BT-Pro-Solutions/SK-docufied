# Explanation

This C# code file provides examples of how to use the Azure OpenAI chat completion API with reasoning features, utilizing the Semantic Kernel SDK. It contains a test class with two asynchronous test methods:

1. **ChatPromptWithReasoningAsync:**  
   Demonstrates using a `Kernel` with Azure OpenAI chat completion. It builds a chat prompt using Semantic Kernel's prompt syntax, sets execution settings to enable reasoning (currently available for certain OpenAI models), and invokes the prompt to receive a chat-based response tailored to expert .NET programming using Semantic Kernel.

2. **ServicePromptWithReasoningAsync:**  
   Shows how to interact directly with the `IChatCompletionService` by managing a `ChatHistory`. Similarly, it configures reasoning effort and retrieves a completion for a multi-turn chat scenario.

Key features:
- Shows how to set execution settings, such as max tokens and reasoning effort, which influences the quality and depth of model reasoning.
- Demonstrates integration points with both the higher-level Semantic Kernel APIs and the direct chat service interface.
- Useful for developers wanting sample code to implement chatbots or conversational agents with advanced reasoning using Azure OpenAI and Microsoft's Semantic Kernel SDK.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.AzureOpenAI;
using OpenAI.Chat;

namespace ChatCompletion;

/// <summary>
/// These examples demonstrate different ways of using chat completion reasoning models with Azure OpenAI API.
/// </summary>
public class AzureOpenAI_ChatCompletionWithReasoning(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Sample showing how to use <see cref="Kernel"/> with chat completion and chat prompt syntax.
    /// </summary>
    [Fact]
    public async Task ChatPromptWithReasoningAsync()
    {
        Console.WriteLine("======== Azure Open AI - Chat Completion with Reasoning ========");

        Assert.NotNull(TestConfiguration.AzureOpenAI.ChatDeploymentName);
        Assert.NotNull(TestConfiguration.AzureOpenAI.Endpoint);
        Assert.NotNull(TestConfiguration.AzureOpenAI.ApiKey);

        var kernel = Kernel.CreateBuilder()
            .AddAzureOpenAIChatCompletion(
                deploymentName: TestConfiguration.AzureOpenAI.ChatDeploymentName,
                endpoint: TestConfiguration.AzureOpenAI.Endpoint,
                apiKey: TestConfiguration.AzureOpenAI.ApiKey,
                modelId: TestConfiguration.AzureOpenAI.ChatModelId)
            .Build();

        // Create execution settings with high reasoning effort.
        var executionSettings = new AzureOpenAIPromptExecutionSettings //OpenAIPromptExecutionSettings
        {
            // Flags Azure SDK to use the new token property.
            SetNewMaxCompletionTokensEnabled = true,
            MaxTokens = 2000,
            // Note: reasoning effort is only available for reasoning models (at this moment o3-mini & o1 models)
            ReasoningEffort = ChatReasoningEffortLevel.Low
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
        Console.WriteLine("======== Azure Open AI - Chat Completion with Azure Default Credential with Reasoning ========");

        Assert.NotNull(TestConfiguration.AzureOpenAI.ChatDeploymentName);
        Assert.NotNull(TestConfiguration.AzureOpenAI.Endpoint);
        Assert.NotNull(TestConfiguration.AzureOpenAI.ApiKey);

        IChatCompletionService chatCompletionService = new AzureOpenAIChatCompletionService(
            deploymentName: TestConfiguration.AzureOpenAI.ChatDeploymentName,
            endpoint: TestConfiguration.AzureOpenAI.Endpoint,
            apiKey: TestConfiguration.AzureOpenAI.ApiKey,
            modelId: TestConfiguration.AzureOpenAI.ChatModelId);

        // Create execution settings with high reasoning effort.
        var executionSettings = new AzureOpenAIPromptExecutionSettings
        {
            // Flags Azure SDK to use the new token property.
            SetNewMaxCompletionTokensEnabled = true,
            MaxTokens = 2000,
            // Note: reasoning effort is only available for reasoning models (at this moment o3-mini & o1 models)
            ReasoningEffort = ChatReasoningEffortLevel.Low
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