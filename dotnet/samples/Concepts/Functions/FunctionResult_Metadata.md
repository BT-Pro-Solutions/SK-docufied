# Explanation

This file implements a set of unit tests using xUnit to demonstrate how to retrieve and display metadata resulting from function invocations in the Microsoft Semantic Kernel framework, specifically with OpenAI chat completions. The three tests in the `FunctionResult_Metadata` class show how to:

- Define and invoke inline functions with the kernel.
- Retrieve the value and usage metadata (`Usage`) from a function result.
- Retrieve the full model metadata from a function result.
- Access and display metadata while streaming function results.

The file expects relevant configuration like API key and model ID for OpenAI services to be provided via `TestConfiguration`. The tests are designed to print out both the primary output (e.g., text completion) and associated metadata in JSON form, illustrating how to handle and inspect result metadata in Semantic Kernel workflows.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

namespace Functions;

public class FunctionResult_Metadata(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task GetTokenUsageMetadataAsync()
    {
        Console.WriteLine("======== Inline Function Definition + Invocation ========");

        // Create kernel
        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey)
            .Build();

        // Create function
        const string FunctionDefinition = "Hi, give me 5 book suggestions about: {{$input}}";
        KernelFunction myFunction = kernel.CreateFunctionFromPrompt(FunctionDefinition);

        // Invoke function through kernel
        FunctionResult result = await kernel.InvokeAsync(myFunction, new() { ["input"] = "travel" });

        // Display results
        Console.WriteLine(result.GetValue<string>());
        Console.WriteLine(result.Metadata?["Usage"]?.AsJson());
        Console.WriteLine();
    }

    [Fact]
    public async Task GetFullModelMetadataAsync()
    {
        Console.WriteLine("======== Inline Function Definition + Invocation ========");

        // Create kernel
        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey)
            .Build();

        // Create function
        const string FunctionDefinition = "1 + 1 = ?";
        KernelFunction myFunction = kernel.CreateFunctionFromPrompt(FunctionDefinition);

        // Invoke function through kernel
        FunctionResult result = await kernel.InvokeAsync(myFunction);

        // Display results
        Console.WriteLine(result.GetValue<string>());
        Console.WriteLine(result.Metadata?.AsJson());
        Console.WriteLine();
    }

    [Fact]
    public async Task GetMetadataFromStreamAsync()
    {
        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey)
            .Build();

        // Create function
        const string FunctionDefinition = "1 + 1 = ?";
        KernelFunction myFunction = kernel.CreateFunctionFromPrompt(FunctionDefinition);

        await foreach (var content in kernel.InvokeStreamingAsync(myFunction))
        {
            Console.WriteLine(content.Metadata?.AsJson());
        }
    }
}
```