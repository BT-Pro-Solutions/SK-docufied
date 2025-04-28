# Explanation
This C# code defines a `SummaryAgentHttpClient` class that uses an injected `HttpClient` to send a text summarization request to a backend API endpoint. The class contains an asynchronous method `SummarizeAsync` that takes a string to summarize, serializes it into JSON, sends it via a POST request to the `/api/summary` endpoint, and returns the summarized text response. It ensures the HTTP response is successful before reading and returning the response content.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text;
using System.Text.Json;
using ProcessFramework.Aspire.Shared;

namespace ProcessFramework.Aspire.ProcessOrchestrator;

public class SummaryAgentHttpClient(HttpClient httpClient)
{
    public async Task<string> SummarizeAsync(string textToSummarize)
    {
        var payload = new SummarizeRequest { TextToSummarize = textToSummarize };
#pragma warning disable CA2234 // We cannot pass uri here since we are using a customer http client with a base address
        var response = await httpClient.PostAsync("/api/summary", new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json")).ConfigureAwait(false);
        response.EnsureSuccessStatusCode();
        var responseContent = await response.Content.ReadAsStringAsync();
        return responseContent;
    }
}
```
