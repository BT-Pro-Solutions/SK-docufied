# Explanation

This C# file demonstrates how to instantiate and configure a Semantic Kernel using the KernelBuilder class from the Microsoft.SemanticKernel library. It provides example test methods showing two different kernel configurations: one with Azure OpenAI chat completion service and another with the inclusion of plugins (specifically an HTTP Plugin). The code is structured as a test class using the [Fact] attribute, illustrating best practices for easy and modular kernel setup in .NET applications that use semantic AI capabilities.

```csharp
// Copyright (c) Microsoft. All rights reserved.

// ==========================================================================================================
// The easier way to instantiate the Semantic Kernel is to use KernelBuilder.
// You can access the builder using Kernel.CreateBuilder().

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Plugins.Core;

namespace KernelExamples;

public class BuildingKernel(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public void BuildKernelWithAzureChatCompletion()
    {
        // KernelBuilder provides a simple way to configure a Kernel. This constructs a kernel
        // with logging and an Azure OpenAI chat completion service configured.
        Kernel kernel1 = Kernel.CreateBuilder()
            .AddAzureOpenAIChatCompletion(
                deploymentName: TestConfiguration.AzureOpenAI.ChatDeploymentName,
                endpoint: TestConfiguration.AzureOpenAI.Endpoint,
                apiKey: TestConfiguration.AzureOpenAI.ApiKey,
                modelId: TestConfiguration.AzureOpenAI.ChatModelId)
            .Build();
    }

    [Fact]
    public void BuildKernelWithPlugins()
    {
        // Plugins may also be configured via the corresponding Plugins property.
        var builder = Kernel.CreateBuilder();
        builder.Plugins.AddFromType<HttpPlugin>();
        Kernel kernel3 = builder.Build();
    }
}
```