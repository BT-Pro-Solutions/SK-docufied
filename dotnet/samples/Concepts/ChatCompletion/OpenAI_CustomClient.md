# Explanation
This C# code provides an example of customizing HTTP requests made to the OpenAI API by using a custom HttpClient and HttpClientHandler. The main class, `OpenAI_CustomClient`, demonstrates how to:
- Inject custom headers into HTTP requests,
- Capture and output request URIs and headers (excluding authorization headers),
- Configure the OpenAI client to use the custom HttpClient,
- Integrate this OpenAI client with the Microsoft Semantic Kernel for plugin-style chat completion,
- Add a sample semantic "FunPlugin" and invoke a function (e.g., generating excuses) using the configured OpenAI client.

The nested `MyCustomClientHttpHandler` class outputs the request URI and all headers (except "Authorization") to a provided test output helper, making this example useful for debugging or diagnostic purposes.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ClientModel;
using System.ClientModel.Primitives;
using Microsoft.SemanticKernel;
using OpenAI;

#pragma warning disable CA5399 // HttpClient is created without enabling CheckCertificateRevocationList

namespace ChatCompletion;

/// <summary>
/// This example shows a way of using a Custom HttpClient and HttpHandler with OpenAI Connector to capture
/// the request Uri and Headers for each request.
/// </summary>
public sealed class OpenAI_CustomClient(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task UsingCustomHttpClientWithOpenAI()
    {
        Assert.NotNull(TestConfiguration.OpenAI.ChatModelId);
        Assert.NotNull(TestConfiguration.OpenAI.ApiKey);

        Console.WriteLine($"======== Open AI - {nameof(UsingCustomHttpClientWithOpenAI)} ========");

        // Create an HttpClient and include your custom header(s)
        using var myCustomHttpHandler = new MyCustomClientHttpHandler(Output);
        using var myCustomClient = new HttpClient(handler: myCustomHttpHandler);
        myCustomClient.DefaultRequestHeaders.Add("My-Custom-Header", "My Custom Value");

        // Configure AzureOpenAIClient to use the customized HttpClient
        var clientOptions = new OpenAIClientOptions
        {
            Transport = new HttpClientPipelineTransport(myCustomClient),
            NetworkTimeout = TimeSpan.FromSeconds(30),
            RetryPolicy = new ClientRetryPolicy()
        };

        var customClient = new OpenAIClient(new ApiKeyCredential(TestConfiguration.OpenAI.ApiKey), clientOptions);

        var kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(TestConfiguration.OpenAI.ChatModelId, customClient)
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