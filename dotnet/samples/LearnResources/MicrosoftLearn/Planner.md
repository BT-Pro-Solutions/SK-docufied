# Explanation

This C# code demonstrates how to set up a conversational AI planner using Microsoft's Semantic Kernel along with Azure OpenAI services. The `Planner` class creates a chat interface where the agent can process user inputs, utilize native functions (such as a `MathSolver` plugin), and reply in a loop. It shows how to build the kernel, add plugins, log interactions, manage chat history, and enable AI function calling with the OpenAI chat completion service. The user interacts with the assistant via the console, while the AI streams its response and can call native code as required.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Plugins;

namespace Examples;

/// <summary>
/// This example demonstrates how to create native functions for AI to call as described at
/// https://learn.microsoft.com/semantic-kernel/agents/plugins/using-the-KernelFunction-decorator
/// </summary>
public class Planner(ITestOutputHelper output) : LearnBaseTest(output)
{
    [Fact]
    public async Task RunAsync()
    {
        Console.WriteLine("======== Planner ========");

        string? endpoint = TestConfiguration.AzureOpenAI.Endpoint;
        string? modelId = TestConfiguration.AzureOpenAI.ChatModelId;
        string? apiKey = TestConfiguration.AzureOpenAI.ApiKey;

        if (endpoint is null || modelId is null || apiKey is null)
        {
            Console.WriteLine("Azure OpenAI credentials not found. Skipping example.");

            return;
        }

        // <RunningNativeFunction>
        var builder = Kernel.CreateBuilder()
                            .AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
        builder.Services.AddLogging(c => c.AddDebug().SetMinimumLevel(LogLevel.Trace));
        builder.Plugins.AddFromType<MathSolver>();
        Kernel kernel = builder.Build();

        // Get chat completion service
        var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

        // Create chat history
        ChatHistory history = [];

        // Start the conversation
        Console.Write("User > ");
        string? userInput;
        while ((userInput = Console.ReadLine()) is not null)
        {
            // Get user input
            Console.Write("User > ");
            history.AddUserMessage(userInput!);

            // Enable auto function calling
            OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new()
            {
                FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
            };

            // Get the response from the AI
            var result = chatCompletionService.GetStreamingChatMessageContentsAsync(
                                history,
                                executionSettings: openAIPromptExecutionSettings,
                                kernel: kernel);

            // Stream the results
            string fullMessage = "";
            var first = true;
            await foreach (var content in result)
            {
                if (content.Role.HasValue && first)
                {
                    Console.Write("Assistant > ");
                    first = false;
                }
                Console.Write(content.Content);
                fullMessage += content.Content;
            }
            Console.WriteLine();

            // Add the message from the agent to the chat history
            history.AddAssistantMessage(fullMessage);

            // Get user input again
            Console.Write("User > ");
        }
    }
}
```