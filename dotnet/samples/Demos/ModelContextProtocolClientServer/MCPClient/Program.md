# Explanation
This C# program serves as the entry point for running a series of sample applications related to MCPClient. It sequentially executes asynchronous sample methods demonstrating different features or capabilities within the MCPClient ecosystem, including tools, prompts, resources, resource templates, sampling, chat completion agents, Azure AI agents, and agent tools.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Threading.Tasks;
using MCPClient.Samples;

namespace MCPClient;

internal sealed class Program
{
    /// <summary>
    /// Main method to run all the samples.
    /// </summary>
    public static async Task Main(string[] args)
    {
        await MCPToolsSample.RunAsync();

        await MCPPromptSample.RunAsync();

        await MCPResourcesSample.RunAsync();

        await MCPResourceTemplatesSample.RunAsync();

        await MCPSamplingSample.RunAsync();

        await ChatCompletionAgentWithMCPToolsSample.RunAsync();

        await AzureAIAgentWithMCPToolsSample.RunAsync();

        await AgentAvailableAsMCPToolSample.RunAsync();
    }
}
```
