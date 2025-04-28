# Explanation
This C# code demonstrates how to use a custom `HttpClient` and `HttpHandler` with the Azure OpenAI Connector to capture request URIs and headers for each HTTP request. The test method `UsingCustomHttpClientWithAzureOpenAI` configures a specialized `HttpClient` with a custom header and routes all requests through a custom handler that logs relevant request details. This custom handler, `MyCustomClientHttpHandler`, uses an `ITestOutputHelper` to output the requested URI and headers (except authorization headers) for each outgoing request. The example integrates this client into an Azure OpenAI kernel setup, imports a semantic plugin, and invokes a prompt to demonstrate the full flow. This is useful for debugging, auditing, or extending the HTTP communication layer of the Azure OpenAI SDK.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ClientModel;
using System.ClientModel.Primitives;
using Azure.AI.OpenAI;
using Microsoft.SemanticKernel;

#pragma warning disable CA5399 // HttpClient is created without enabling CheckCertificateRevocationList

namespace ChatCompletion;

/// <summary>
/// This example shows a way of using a Custom HttpClient and HttpHandler with Azure OpenAI Connector to capture
/// the request Uri and Headers for each request.
/// </summary>
public sealed class AzureOpenAI_CustomClient(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task UsingCustomHttpClientWithAzureOpenAI()
    {
        Assert.NotNull(TestConfiguration.AzureOpenAI.Endpoint);
        Assert.NotNull(TestConfiguration.AzureOpenAI.ChatDeploymentName);
        Assert.NotNull(TestConfiguration.AzureOpenAI.ApiKey);

        Console.WriteLine($"======== Azure Open AI - {nameof(UsingCustomHttpClientWithAzureOpenAI)} ========");

        // Create an HttpClient and include your custom header(s)
        using var myCustomHttpHandler = new MyCustomClientHttpHandler(Output);
        using var myCustomClient = new HttpClient(handler: myCustomHttpHandler);
        myCustomClient.DefaultRequestHeaders.Add("My-Custom-Header", "My Custom Value");

        // Configure AzureOpenAIClient to use the customized HttpClient
        var clientOptions = new AzureOpenAIClientOptions
        {
            Transport = new HttpClientPipelineTransport(myCustomClient),
            NetworkTimeout = TimeSpan.FromSeconds(30),
            RetryPolicy = new ClientRetryPolicy()
        };

        var customClient = new AzureOpenAIClient(new Uri(TestConfiguration.AzureOpenAI.Endpoint), new ApiKeyCredential(TestConfiguration.AzureOpenAI.ApiKey), clientOptions);

        var kernel = Kernel.CreateBuilder()
            .AddAzureOpenAIChatCompletion(TestConfiguration.AzureOpenAI.ChatDeploymentName, customClient)
            .Build();

        // Load semantic plugin defined with prompt templates
        string folder = RepoFiles.SamplePluginsPath();

        kernel.ImportPluginFromPromptDirectory(Path.Combine(folder, "FunPlugin"));

        // Run
        var result = await kernel.InvokeAsync(
            kernel.Plugins["FunPlugin"]["Excuses"],
            new() { ["input"] = "I have no homework" }
        );
        Console.WriteLine(result.GetValue<string>());

        myCustomClient.Dispose();
    }

    /// <summary>
    /// Normally you would use a custom HttpClientHandler to add custom logic to your custom http client
    /// This uses the ITestOutputHelper to write the requested URI to the test output
    /// </summary>
    /// <param name="output">The <see cref="ITestOutputHelper"/> to write the requested URI to the test output </param>
    private sealed class MyCustomClientHttpHandler(ITestOutputHelper output) : HttpClientHandler
    {
        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            output.WriteLine($"Requested URI: {request.RequestUri}");

            request.Headers.Where(h => h.Key != "Authorization")
                .ToList()
                .ForEach(h => output.WriteLine($"{h.Key}: {string.Join(", ", h.Value)}"));
            output.WriteLine("--------------------------------");

            // Add custom logic here
            return await base.SendAsync(request, cancellationToken);
        }
    }
}
```