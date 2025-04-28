# Explanation
This C# code demonstrates how to use the Microsoft Semantic Kernel to construct and invoke a chat prompt using OpenAI's chat completion API. It defines a test class `Step5_Chat_Prompt` with a test method `InvokeChatPrompt` that creates a Semantic Kernel configured for OpenAI, constructs a chat prompt with user and system messages, invokes it asynchronously, and prints the result. The system prompt instructs the response to be in JSON format, and the kernel returns the OpenAI completion based on that prompt.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

namespace GettingStarted;

public sealed class Step5_Chat_Prompt(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Show how to construct a chat prompt and invoke it.
    /// </summary>
    [Fact]
    public async Task InvokeChatPrompt()
    {
        // Create a kernel with OpenAI chat completion
        Kernel kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey)
            .Build();

        // Invoke the kernel with a chat prompt and display the result
        string chatPrompt = """
            <message role="user">What is Seattle?</message>
            <message role="system">Respond with JSON.</message>
            """;

        Console.WriteLine(await kernel.InvokePromptAsync(chatPrompt));
    }
}
```