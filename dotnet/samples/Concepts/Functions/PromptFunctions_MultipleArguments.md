# Explanation

This C# code demonstrates how to use the Microsoft Semantic Kernel to invoke a C# method function that accepts multiple arguments from within a prompt template written in natural language. Specifically, the example loads a native plugin (`TextPlugin`) that provides a `Concat` method—allowing text concatenation—which can then be accessed from prompt templates as `text.Concat`. The example shows how arguments can be passed from the context (including variables) to these method functions via named arguments in a prompt template. The rendered natural language prompt (produced by combining arguments with the method function) is then sent to an Azure OpenAI endpoint using the Semantic Kernel's orchestration, and the result (a generated haiku) is displayed. The code is intended as an integration test/example for using prompt functions with multiple arguments inside the Semantic Kernel framework.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.Plugins.Core;

namespace Functions;

public class PromptFunctions_MultipleArguments(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Show how to invoke a Method Function written in C# with multiple arguments
    /// from a Prompt Function written in natural language
    /// </summary>
    [Fact]
    public async Task RunAsync()
    {
        Console.WriteLine("======== TemplateMethodFunctionsWithMultipleArguments ========");

        string serviceId = TestConfiguration.AzureOpenAI.ServiceId;
        string apiKey = TestConfiguration.AzureOpenAI.ApiKey;
        string deploymentName = TestConfiguration.AzureOpenAI.ChatDeploymentName;
        string modelId = TestConfiguration.AzureOpenAI.ChatModelId;
        string endpoint = TestConfiguration.AzureOpenAI.Endpoint;

        if (apiKey is null || deploymentName is null || modelId is null || endpoint is null)
        {
            Console.WriteLine("AzureOpenAI modelId, endpoint, apiKey, or deploymentName not found. Skipping example.");
            return;
        }

        IKernelBuilder builder = Kernel.CreateBuilder();
        builder.Services.AddLogging(c => c.AddConsole());
        builder.AddAzureOpenAIChatCompletion(
            deploymentName: deploymentName,
            endpoint: endpoint,
            serviceId: serviceId,
            apiKey: apiKey,
            modelId: modelId);
        Kernel kernel = builder.Build();

        var arguments = new KernelArguments
        {
            ["word2"] = " Potter"
        };

        // Load native plugin into the kernel function collection, sharing its functions with prompt templates
        // Functions loaded here are available as "text.*"
        kernel.ImportPluginFromType<TextPlugin>("text");

        // Prompt Function invoking text.Concat method function with named arguments input and input2 where input is a string and input2 is set to a variable from context called word2.
        const string FunctionDefinition = @"
 Write a haiku about the following: {{text.Concat input='Harry' input2=$word2}}
";

        // This allows to see the prompt before it's sent to OpenAI
        Console.WriteLine("--- Rendered Prompt");
        var promptTemplateFactory = new KernelPromptTemplateFactory();
        var promptTemplate = promptTemplateFactory.Create(new PromptTemplateConfig(FunctionDefinition));
        var renderedPrompt = await promptTemplate.RenderAsync(kernel, arguments);
        Console.WriteLine(renderedPrompt);

        // Run the prompt / prompt function
        var haiku = kernel.CreateFunctionFromPrompt(FunctionDefinition, new OpenAIPromptExecutionSettings() { MaxTokens = 100 });

        // Show the result
        Console.WriteLine("--- Prompt Function result");
        var result = await kernel.InvokeAsync(haiku, arguments);
        Console.WriteLine(result.GetValue<string>());

        /* OUTPUT:

--- Rendered Prompt

 Write a haiku about the following: Harry Potter

--- Prompt Function result
A boy with a scar,
Wizarding world he explores,
Harry Potter's tale.
         */
    }
}
```
