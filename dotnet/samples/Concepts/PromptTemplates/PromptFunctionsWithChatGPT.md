# Explanation

This code demonstrates how to use the Azure OpenAI GPT-3.5 Chat model for prompt-based text generation within a .NET project using the Microsoft Semantic Kernel SDK. It sets up a `Kernel` configured for Azure OpenAI, creates a prompt function to list two planets nearest to a specified planet (excluding moons), and invokes this function asynchronously. After generating the response based on the input ("Jupiter"), the result is printed to the console. The code is organized as a unit test case compatible with xUnit's `[Fact]` attribute.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

namespace PromptTemplates;

/// <summary>
/// This example shows how to use GPT3.5 Chat model for prompts and prompt functions.
/// </summary>
public class PromptFunctionsWithChatGPT(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task RunAsync()
    {
        Console.WriteLine("======== Using Chat GPT model for text generation ========");

        Kernel kernel = Kernel.CreateBuilder()
            .AddAzureOpenAIChatCompletion(
                deploymentName: TestConfiguration.AzureOpenAI.ChatDeploymentName,
                endpoint: TestConfiguration.AzureOpenAI.Endpoint,
                apiKey: TestConfiguration.AzureOpenAI.ApiKey,
                modelId: TestConfiguration.AzureOpenAI.ChatModelId)
            .Build();

        var func = kernel.CreateFunctionFromPrompt(
            "List the two planets closest to '{{$input}}', excluding moons, using bullet points.");

        var result = await func.InvokeAsync(kernel, new() { ["input"] = "Jupiter" });
        Console.WriteLine(result.GetValue<string>());

        /*
        Output:
           - Saturn
           - Uranus
        */
    }
}
```
