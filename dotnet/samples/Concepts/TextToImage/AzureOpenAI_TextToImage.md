# Explanation

This C# code file demonstrates how to leverage the Microsoft Semantic Kernel with Azure OpenAI's DALL-E for text-to-image generation. It contains a class, `AzureOpenAI_TextToImage`, which provides two unit tests: one for generating an image and retrieving its URI, and another for generating an image and retrieving its binary data. The examples show how to configure the kernel for Azure OpenAI, use the image generation service, and access results. The code integrates configuration for endpoints, API keys, and model identities, and logs output for verification. It is primarily intended for integration testing and learning how to use Semantic Kernel's image generation capabilities with Azure's DALL-E service.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.TextToImage;

namespace TextToImage;

// The following example shows how to use Semantic Kernel with OpenAI DALL-E 2 to create images
public class AzureOpenAI_TextToImage(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task SimpleDallE3ImageUriAsync()
    {
        var builder = Kernel.CreateBuilder()
           .AddAzureOpenAITextToImage( // Add your text to image service
               deploymentName: TestConfiguration.AzureOpenAI.ImageDeploymentName,
               endpoint: TestConfiguration.AzureOpenAI.ImageEndpoint,
               apiKey: TestConfiguration.AzureOpenAI.ImageApiKey,
               modelId: TestConfiguration.AzureOpenAI.ImageModelId);

        var kernel = builder.Build();
        var service = kernel.GetRequiredService<ITextToImageService>();

        var generatedImages = await service.GetImageContentsAsync(
            new TextContent("A cute baby sea otter"),
            new OpenAITextToImageExecutionSettings { Size = (Width: 1792, Height: 1024) });

        this.Output.WriteLine(generatedImages[0].Uri!.ToString());
    }

    [Fact]
    public async Task SimpleDallE3ImageBinaryAsync()
    {
        var builder = Kernel.CreateBuilder()
           .AddAzureOpenAITextToImage( // Add your text to image service
               deploymentName: TestConfiguration.AzureOpenAI.ImageDeploymentName,
               endpoint: TestConfiguration.AzureOpenAI.ImageEndpoint,
               apiKey: TestConfiguration.AzureOpenAI.ImageApiKey,
               modelId: TestConfiguration.AzureOpenAI.ImageModelId);

        var kernel = builder.Build();
        var service = kernel.GetRequiredService<ITextToImageService>();

        var generatedImages = await service.GetImageContentsAsync(new TextContent("A cute baby sea otter"),
            new OpenAITextToImageExecutionSettings
            {
                Size = (Width: 1024, Height: 1024),

                // Response Format also accepts the OpenAI.Images.GeneratedImageFormat type. 
                ResponseFormat = "bytes",
            });

        this.Output.WriteLine($"Generated Image Bytes: {generatedImages[0].Data!.Value.Length}");
        this.Output.WriteLine($"Generated Image DataUri: {generatedImages[0].DataUri}");
    }
}
```
