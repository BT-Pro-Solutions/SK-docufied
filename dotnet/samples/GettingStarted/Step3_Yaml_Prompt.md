# Explanation

This C# code demonstrates how to create and use a prompt `KernelFunction` from a YAML resource within the Microsoft Semantic Kernel framework. The example defines a test class `Step3_Yaml_Prompt` that shows how to load a YAML-formatted prompt from embedded resources, instantiate the function using the Semantic Kernel API, and invoke it with arguments. It also demonstrates how to use both the standard YAML prompt and a template using Handlebars syntax for prompt creation. The results are printed to the console for verification. This approach is useful when you want to separate prompt definitions from your main code, storing them as resources and flexibly instantiating them at runtime.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.PromptTemplates.Handlebars;
using Resources;

namespace GettingStarted;

/// <summary>
/// This example shows how to create a prompt <see cref="KernelFunction"/> from a YAML resource.
/// </summary>
public sealed class Step3_Yaml_Prompt(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Show how to create a prompt <see cref="KernelFunction"/> from a YAML resource.
    /// </summary>
    [Fact]
    public async Task CreatePromptFromYaml()
    {
        // Create a kernel with OpenAI chat completion
        Kernel kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey)
            .Build();

        // Load prompt from resource
        var generateStoryYaml = EmbeddedResource.Read("GenerateStory.yaml");
        var function = kernel.CreateFunctionFromPromptYaml(generateStoryYaml);

        // Invoke the prompt function and display the result
        Console.WriteLine(await kernel.InvokeAsync(function, arguments: new()
            {
                { "topic", "Dog" },
                { "length", "3" },
            }));

        // Load prompt from resource
        var generateStoryHandlebarsYaml = EmbeddedResource.Read("GenerateStoryHandlebars.yaml");
        function = kernel.CreateFunctionFromPromptYaml(generateStoryHandlebarsYaml, new HandlebarsPromptTemplateFactory());

        // Invoke the prompt function and display the result
        Console.WriteLine(await kernel.InvokeAsync(function, arguments: new()
            {
                { "topic", "Cat" },
                { "length", "3" },
            }));
    }
}
```
