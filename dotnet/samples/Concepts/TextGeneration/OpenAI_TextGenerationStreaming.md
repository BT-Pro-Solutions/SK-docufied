# Explanation

This C# code demonstrates how to use Semantic Kernel for streaming text generation with both Azure OpenAI and OpenAI's text generation models. The class defines two test methods, each setting up a different text generation service (AzureOpenAIChatCompletionService and OpenAIChatCompletionService) for streaming responses. It should be noted that this example is specifically for text completion models (not chat completions), and that such models are deprecated and will be removed in the future. The core logic streams the output for a prompt ("Write one paragraph why AI is awesome") and prints the response as it is received.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel.Connectors.AzureOpenAI;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.TextGeneration;

namespace TextGeneration;

/**
 * The following example shows how to use Semantic Kernel with streaming text generation.
 *
 * This example will NOT work with regular chat completion models. It will only work with
 * text completion models.
 *
 * Note that all text generation models are deprecated by OpenAI and will be removed in a future release.
 *
 * Refer to example 33 for streaming chat completion.
 */
public class OpenAI_TextGenerationStreaming(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public Task AzureOpenAITextGenerationStreamAsync()
    {
        Console.WriteLine("======== Azure OpenAI - Text Generation - Raw Streaming ========");

        var textGeneration = new AzureOpenAIChatCompletionService(
            deploymentName: TestConfiguration.AzureOpenAI.ChatDeploymentName,
            endpoint: TestConfiguration.AzureOpenAI.Endpoint,
            apiKey: TestConfiguration.AzureOpenAI.ApiKey,
            modelId: TestConfiguration.AzureOpenAI.ChatModelId);

        return this.TextGenerationStreamAsync(textGeneration);
    }

    [Fact]
    public Task OpenAITextGenerationStreamAsync()
    {
        Console.WriteLine("======== Open AI - Text Generation - Raw Streaming ========");

        var textGeneration = new OpenAIChatCompletionService(TestConfiguration.OpenAI.ChatModelId, TestConfiguration.OpenAI.ApiKey);

        return this.TextGenerationStreamAsync(textGeneration);
    }

    private async Task TextGenerationStreamAsync(ITextGenerationService textGeneration)
    {
        var executionSettings = new OpenAIPromptExecutionSettings()
        {
            MaxTokens = 100,
            FrequencyPenalty = 0,
            PresencePenalty = 0,
            Temperature = 1,
            TopP = 0.5
        };

        var prompt = "Write one paragraph why AI is awesome";

        Console.WriteLine("Prompt: " + prompt);
        await foreach (var content in textGeneration.GetStreamingTextContentsAsync(prompt, executionSettings))
        {
            Console.Write(content);
        }

        Console.WriteLine();
    }
}
```