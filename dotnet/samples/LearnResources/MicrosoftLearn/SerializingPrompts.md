# Explanation

This C# code file demonstrates how to serialize and use prompts with the Microsoft Semantic Kernel SDK. It provides an example of loading prompts from files, as described in the Microsoft documentation on saving prompts as files. The code creates a chat loop that utilizes a loaded YAML prompt (for intent detection) and another prompt for chat responses, integrating few-shot examples to guide the prompt engine.

The workflow involves:
- Configuring an Azure OpenAI endpoint and loading plugin prompts from a directory.
- Loading a specific YAML prompt from embedded resources.
- Defining few-shot examples and choices to help guide the intent classification.
- Running a command-line chat loop, where each user input is analyzed for intent (to continue or end the conversation), and then generates a response with streaming output.
- Maintaining and updating conversation history throughout the session.

The code assumes the existence of certain resources and configuration settings, including plugin prompt files and Azure OpenAI credentials.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Reflection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Plugins.Core;
using Microsoft.SemanticKernel.PromptTemplates.Handlebars;

namespace Examples;

/// <summary>
/// This example demonstrates how to serialize prompts as described at
/// https://learn.microsoft.com/semantic-kernel/prompts/saving-prompts-as-files
/// </summary>
public class SerializingPrompts(ITestOutputHelper output) : LearnBaseTest([
            "Can you send an approval to the marketing team?",
    "That is all, thanks."], output)
{
    [Fact]
    public async Task RunAsync()
    {
        Console.WriteLine("======== Serializing Prompts ========");

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

        // Load prompts
        var prompts = kernel.CreatePluginFromPromptDirectory("./../../../Plugins/Prompts");

        // Load prompt from YAML
        using StreamReader reader = new(Assembly.GetExecutingAssembly().GetManifestResourceStream("Resources.getIntent.prompt.yaml")!);
        KernelFunction getIntent = kernel.CreateFunctionFromPromptYaml(
            await reader.ReadToEndAsync(),
            promptTemplateFactory: new HandlebarsPromptTemplateFactory()
        );

        // Create choices
        List<string> choices = ["ContinueConversation", "EndConversation"];

        // Create few-shot examples
        List<ChatHistory> fewShotExamples =
        [
            [
                new ChatMessageContent(AuthorRole.User, "Can you send a very quick approval to the marketing team?"),
                new ChatMessageContent(AuthorRole.System, "Intent:"),
                new ChatMessageContent(AuthorRole.Assistant, "ContinueConversation")
            ],
            [
                new ChatMessageContent(AuthorRole.User, "Can you send the full update to the marketing team?"),
                new ChatMessageContent(AuthorRole.System, "Intent:"),
                new ChatMessageContent(AuthorRole.Assistant, "EndConversation")
            ]
        ];

        // Create chat history
        ChatHistory history = [];

        // Start the chat loop
        Console.Write("User > ");
        string? userInput;
        while ((userInput = Console.ReadLine()) is not null)
        {
            // Invoke handlebars prompt
            var intent = await kernel.InvokeAsync(
                getIntent,
                new()
                {
                    { "request", userInput },
                    { "choices", choices },
                    { "history", history },
                    { "fewShotExamples", fewShotExamples }
                }
            );

            // End the chat if the intent is "Stop"
            if (intent.ToString() == "EndConversation")
            {
                break;
            }

            // Get chat response
            var chatResult = kernel.InvokeStreamingAsync<StreamingChatMessageContent>(
                prompts["chat"],
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