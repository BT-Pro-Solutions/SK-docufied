# Explanation

This C# code demonstrates how to use the OpenAI GPT-4 Vision model with Microsoft's Semantic Kernel library for chat completion tasks involving both text and images. It provides three test cases:

1. **RemoteImageAsync**: Sends a user message containing a text prompt and a remote image URL to the chat model, then prints the AI's description of the image.
2. **LocalImageAsync**: Reads an embedded local image as bytes, constructs a message combining text and the image, and outputs the AI's response.
3. **LocalImageWithImageDetailInMetadataAsync**: Similar to the previous test, but adds image metadata specifying a high detail level for analysis.

These examples illustrate multi-modal input handling (text + image) and how to pass images and associated metadata to the chat completion service using Semantic Kernel.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Resources;

namespace ChatCompletion;

// This example shows how to use GPT Vision model with different content types (text and image).
public class OpenAI_ChatCompletionWithVision(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task RemoteImageAsync()
    {
        const string ImageUri = "https://upload.wikimedia.org/wikipedia/commons/d/d5/Half-timbered_mansion%2C_Zirkel%2C_East_view.jpg";

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion("gpt-4-vision-preview", TestConfiguration.OpenAI.ApiKey)
            .Build();

        var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

        var chatHistory = new ChatHistory("You are a friendly assistant.");

        chatHistory.AddUserMessage(
        [
            new TextContent("What’s in this image?"),
            new ImageContent(new Uri(ImageUri))
        ]);

        var reply = await chatCompletionService.GetChatMessageContentAsync(chatHistory);

        Console.WriteLine(reply.Content);
    }

    [Fact]
    public async Task LocalImageAsync()
    {
        var imageBytes = await EmbeddedResource.ReadAllAsync("sample_image.jpg");

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion("gpt-4-vision-preview", TestConfiguration.OpenAI.ApiKey)
            .Build();

        var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

        var chatHistory = new ChatHistory("You are a friendly assistant.");

        chatHistory.AddUserMessage(
        [
            new TextContent("What’s in this image?"),
            new ImageContent(imageBytes, "image/jpg")
        ]);

        var reply = await chatCompletionService.GetChatMessageContentAsync(chatHistory);

        Console.WriteLine(reply.Content);
    }

    [Fact]
    public async Task LocalImageWithImageDetailInMetadataAsync()
    {
        var imageBytes = await EmbeddedResource.ReadAllAsync("sample_image.jpg");

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion("gpt-4-vision-preview", TestConfiguration.OpenAI.ApiKey)
            .Build();

        var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

        var chatHistory = new ChatHistory("You are a friendly assistant.");

        chatHistory.AddUserMessage(
        [
            new TextContent("What’s in this image?"),
            new ImageContent(imageBytes, "image/jpg") { Metadata = new Dictionary<string, object?> { ["ChatImageDetailLevel"] = "high" } }
        ]);

        var reply = await chatCompletionService.GetChatMessageContentAsync(chatHistory);

        Console.WriteLine(reply.Content);
    }
}
```