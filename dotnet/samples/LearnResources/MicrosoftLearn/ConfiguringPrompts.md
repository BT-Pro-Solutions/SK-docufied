# Explanation

This C# code demonstrates how to configure prompts using Microsoft's Semantic Kernel SDK in the context of an Azure OpenAI-backed chatbot. The example, intended for use with unit tests and interactive sessions, shows how to:

- Set up a `Kernel` with Azure OpenAI chat completion capability.
- Define a custom function from a prompt template, including settings for different OpenAI models (e.g., `gpt-3.5-turbo`, `gpt-4`).
- Add plugins for conversation summarization.
- Run an interactive chat loop in the console, streaming the assistant's response as the user types.
- Maintain conversation history for context in subsequent turns.

For more context, see [Configure prompts in Semantic Kernel](https://learn.microsoft.com/semantic-kernel/prompts/configure-prompts). This example is useful for developers integrating customized prompt handling and model-specific execution settings within conversational AI workflows.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.Plugins.Core;

namespace Examples;

/// <summary>
/// This example demonstrates how to configure prompts as described at
/// https://learn.microsoft.com/semantic-kernel/prompts/configure-prompts
/// </summary>
public class ConfiguringPrompts(ITestOutputHelper output) : LearnBaseTest(["Who were the Vikings?"], output)
{
    [Fact]
    public async Task RunAsync()
    {
        Console.WriteLine("======== Configuring Prompts ========");

        string? endpoint = TestConfiguration.AzureOpenAI.Endpoint;
        string? modelId = TestConfiguration.AzureOpenAI.ChatModelId;
        string? apiKey = TestConfiguration.AzureOpenAI.ApiKey;

        if (endpoint is null || modelId is null || apiKey is null)
        {
            Console.WriteLine("Azure OpenAI credentials not found. Skipping example.");

            return;
        }

        var builder = Kernel.CreateBuilder()
                            .AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
        builder.Plugins.AddFromType<ConversationSummaryPlugin>();
        Kernel kernel = builder.Build();

        // <FunctionFromPrompt>
        // Create a template for chat with settings
        var chat = kernel.CreateFunctionFromPrompt(
            new PromptTemplateConfig()
            {
                Name = "Chat",
                Description = "Chat with the assistant.",
                Template = @"{{ConversationSummaryPlugin.SummarizeConversation $history}}
                            User: {{$request}}
                            Assistant: ",
                TemplateFormat = "semantic-kernel",
                InputVariables =
                [
                    new() { Name = "history", Description = "The history of the conversation.", IsRequired = false, Default = "" },
                    new() { Name = "request", Description = "The user's request.", IsRequired = true }
                ],
                ExecutionSettings =
                {
                    {
                        "default",
                        new OpenAIPromptExecutionSettings()
                        {
                            MaxTokens = 1000,
                            Temperature = 0
                        }
                    },
                    {
                        "gpt-3.5-turbo", new OpenAIPromptExecutionSettings()
                        {
                            ModelId = "gpt-3.5-turbo-0613",
                            MaxTokens = 4000,
                            Temperature = 0.2
                        }
                    },
                    {
                        "gpt-4",
                        new OpenAIPromptExecutionSettings()
                        {
                            ModelId = "gpt-4-1106-preview",
                            MaxTokens = 8000,
                            Temperature = 0.3
                        }
                    }
                }
            }
        );
        // </FunctionFromPrompt>

        // Create chat history and choices
        ChatHistory history = [];

        // Start the chat loop
        Console.Write("User > ");
        string? userInput;
        while ((userInput = Console.ReadLine()) is not null)
        {
            // Get chat response
            var chatResult = kernel.InvokeStreamingAsync<StreamingChatMessageContent>(
                chat,
                new()
                {
                    { "request", userInput },
                    { "history", string.Join("\n", history.Select(x => x.Role + ": " + x.Content)) }
                }
            );

            // Stream the response
            string message = "";
            await foreach (var chunk in chatResult)
            {
                if (chunk.Role.HasValue)
                {
                    Console.Write(chunk.Role + " > ");
                }
                message += chunk;
                Console.Write(chunk);
            }
            Console.WriteLine();

            // Append to history
            history.AddUserMessage(userInput);
            history.AddAssistantMessage(message);

            // Get user input again
            Console.Write("User > ");
        }
    }
}
```