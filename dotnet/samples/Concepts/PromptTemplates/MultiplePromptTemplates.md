# Explanation
This C# code demonstrates how to use multiple prompt template formats (semantic-kernel, Handlebars, and Liquid) with Microsoft Semantic Kernel. The `MultiplePromptTemplates` class is a test class that shows how to combine different prompt template factories and invoke AI completions using templates in different formats. The class uses retry theory for robust testing and prints out results for each template type. It sets up an Azure OpenAI completion kernel and demonstrates runtime selection of prompt template format, dynamically supplying user arguments (e.g., name) to the prompts.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.PromptTemplates.Handlebars;
using Microsoft.SemanticKernel.PromptTemplates.Liquid;
using xRetry;

namespace PromptTemplates;

// This example shows how to use multiple prompt template formats.
public class MultiplePromptTemplates(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Show how to combine multiple prompt template factories.
    /// </summary>
    [RetryTheory(typeof(HttpOperationException))]
    [InlineData("semantic-kernel", "Hello AI, my name is {{$name}}. What is the origin of my name?", "Paz")]
    [InlineData("handlebars", "Hello AI, my name is {{name}}. What is the origin of my name?", "Mira")]
    [InlineData("liquid", "Hello AI, my name is {{name}}. What is the origin of my name?", "Aoibhinn")]
    public Task InvokeDifferentPromptTypes(string templateFormat, string prompt, string name)
    {
        Console.WriteLine($"======== {nameof(MultiplePromptTemplates)} ========");

        Kernel kernel = Kernel.CreateBuilder()
            .AddAzureOpenAIChatCompletion(
                deploymentName: TestConfiguration.AzureOpenAI.ChatDeploymentName,
                endpoint: TestConfiguration.AzureOpenAI.Endpoint,
                serviceId: "AzureOpenAIChat",
                apiKey: TestConfiguration.AzureOpenAI.ApiKey,
                modelId: TestConfiguration.AzureOpenAI.ChatModelId)
            .Build();

        var promptTemplateFactory = new AggregatorPromptTemplateFactory(
            new KernelPromptTemplateFactory(),
            new HandlebarsPromptTemplateFactory(),
            new LiquidPromptTemplateFactory());

        return RunPromptAsync(kernel, prompt, name, templateFormat, promptTemplateFactory);
    }

    private async Task RunPromptAsync(Kernel kernel, string prompt, string name, string templateFormat, IPromptTemplateFactory promptTemplateFactory)
    {
        Console.WriteLine($"======== {templateFormat} : {prompt} ========");

        var function = kernel.CreateFunctionFromPrompt(
            promptConfig: new PromptTemplateConfig()
            {
                Template = prompt,
                TemplateFormat = templateFormat,
                Name = "MyFunction",
            },
            promptTemplateFactory: promptTemplateFactory
        );

        var arguments = new KernelArguments()
        {
            { "name", name }
        };

        var result = await kernel.InvokeAsync(function, arguments);
        Console.WriteLine(result.GetValue<string>());
    }
}
```