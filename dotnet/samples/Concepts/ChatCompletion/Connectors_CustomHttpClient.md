# Explanation
This C# code demonstrates how to configure and use custom `HttpClient` instances with connectors in the Semantic Kernel (SK) SDK, specifically for OpenAI chat completion APIs. The class contains two test methods:
- `UseDefaultHttpClient` shows how to use the default HttpClient managed by the SK SDK (simply by omitting the `httpMessageInvoker` parameter).
- `UseCustomHttpClient` demonstrates injecting your own `HttpClient` instance for advanced configuration or control (by passing it as an argument).

Both examples rely on settings from a `TestConfiguration` class (such as API keys and model IDs) and are structured as xUnit tests, useful for unit testing scenarios.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

namespace ChatCompletion;

// These examples show how to use a custom HttpClient with SK connectors.
public class Connectors_CustomHttpClient(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Demonstrates the usage of the default HttpClient provided by the SK SDK.
    /// </summary>
    [Fact]
    public void UseDefaultHttpClient()
    {
        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey) // If you need to use the default HttpClient from the SK SDK, simply omit the argument for the httpMessageInvoker parameter.
            .Build();
    }

    /// <summary>
    /// Demonstrates the usage of a custom HttpClient.
    /// </summary>
    [Fact]
    public void UseCustomHttpClient()
    {
        using var httpClient = new HttpClient();

        // If you need to use a custom HttpClient, simply pass it as an argument for the httpClient parameter.
        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey,
                httpClient: httpClient)
            .Build();
    }
}
```
