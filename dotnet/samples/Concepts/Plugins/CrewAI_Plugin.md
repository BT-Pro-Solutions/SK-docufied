# Explanation

This code file demonstrates how to interact with a CrewAI Enterprise Crew, either by invoking it directly or by using it as a plugin within the Microsoft Semantic Kernel ecosystem. The class provides two primary examples:

1. **UsingCrewAIEnterpriseAsync:** Shows how to programmatically kickoff (start) a CrewAI Enterprise Crew with specified inputs, monitor its progress, and await its completion. This requires a valid CrewAI Enterprise deployment, including endpoint and authentication token.

2. **UsingCrewAIEnterpriseAsPluginAsync:** Demonstrates how to configure the CrewAI Enterprise Crew as a plugin. This allows the Crew to be used seamlessly within the Semantic Kernel, using OpenAI services to invoke CrewAI tasks in a more generalized, plugin-driven way. It defines input metadata for the plugin, sets up authentication, creates a plugin, adds it to the kernel, and invokes the crew as a plugin function.

The required inputs and authentication details must be configured in `TestConfiguration`. The code is designed for scenarios where deeper integration with generative AI workflows and Microsoft Semantic Kernel is needed, streamlining invocation and management of enterprise AI tasks.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Plugins.AI.CrewAI;

namespace Plugins;

/// <summary>
/// This example shows how to interact with an existing CrewAI Enterprise Crew directly or as a plugin.
/// These examples require a valid CrewAI Enterprise deployment with an endpoint, auth token, and known inputs.
/// </summary>
public class CrewAI_Plugin(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Shows how to kickoff an existing CrewAI Enterprise Crew and wait for it to complete.
    /// </summary>
    [Fact]
    public async Task UsingCrewAIEnterpriseAsync()
    {
        string crewAIEndpoint = TestConfiguration.CrewAI.Endpoint;
        string crewAIAuthToken = TestConfiguration.CrewAI.AuthToken;

        var crew = new CrewAIEnterprise(
            endpoint: new Uri(crewAIEndpoint),
            authTokenProvider: async () => crewAIAuthToken);

        // The required inputs for the Crew must be known in advance. This example is modeled after the
        // Enterprise Content Marketing Crew Template and requires the following inputs:
        var inputs = new
        {
            company = "CrewAI",
            topic = "Agentic products for consumers",
        };

        // Invoke directly with our inputs
        var kickoffId = await crew.KickoffAsync(inputs);
        Console.WriteLine($"CrewAI Enterprise Crew kicked off with ID: {kickoffId}");

        // Wait for completion
        var result = await crew.WaitForCrewCompletionAsync(kickoffId);
        Console.WriteLine("CrewAI Enterprise Crew completed with the following result:");
        Console.WriteLine(result);
    }

    /// <summary>
    /// Shows how to kickoff an existing CrewAI Enterprise Crew as a plugin.
    /// </summary>
    [Fact]
    public async Task UsingCrewAIEnterpriseAsPluginAsync()
    {
        string crewAIEndpoint = TestConfiguration.CrewAI.Endpoint;
        string crewAIAuthToken = TestConfiguration.CrewAI.AuthToken;
        string openAIModelId = TestConfiguration.OpenAI.ChatModelId;
        string openAIApiKey = TestConfiguration.OpenAI.ApiKey;

        if (openAIModelId is null || openAIApiKey is null)
        {
            Console.WriteLine("OpenAI credentials not found. Skipping example.");
            return;
        }

        // Setup the Kernel and AI Services
        Kernel kernel = Kernel.CreateBuilder()
            .AddOpenAIChatCompletion(
                modelId: openAIModelId,
                apiKey: openAIApiKey)
            .Build();

        var crew = new CrewAIEnterprise(
            endpoint: new Uri(crewAIEndpoint),
            authTokenProvider: async () => crewAIAuthToken);

        // The required inputs for the Crew must be known in advance. This example is modeled after the
        // Enterprise Content Marketing Crew Template and requires string inputs for the company and topic.
        // We need to describe the type and purpose of each input to allow the LLM to invoke the crew as expected.
        var crewPluginDefinitions = new[]
        {
            new CrewAIInputMetadata(Name: "company", Description: "The name of the company that should be researched", Type: typeof(string)),
            new CrewAIInputMetadata(Name: "topic", Description: "The topic that should be researched", Type: typeof(string)),
        };

        // Create the CrewAI Plugin. This builds a plugin that can be added to the Kernel and invoked like any other plugin.
        // The plugin will contain the following functions:
        // - Kickoff: Starts the Crew with the specified inputs and returns the Id of the scheduled kickoff.
        // - KickoffAndWait: Starts the Crew with the specified inputs and waits for the Crew to complete before returning the result.
        // - WaitForCrewCompletion: Waits for the specified Crew kickoff to complete and returns the result.
        // - GetCrewKickoffStatus: Gets the status of the specified Crew kickoff.
        var crewPlugin = crew.CreateKernelPlugin(
            name: "EnterpriseContentMarketingCrew",
            description: "Conducts thorough research on the specified company and topic to identify emerging trends, analyze competitor strategies, and gather data-driven insights.",
            inputMetadata: crewPluginDefinitions);

        // Add the plugin to the Kernel
        kernel.Plugins.Add(crewPlugin);

        // Invoke the CrewAI Plugin directly as shown below, or use automaic function calling with an LLM.
        var kickoffAndWaitFunction = crewPlugin["KickoffAndWait"];
        var result = await kernel.InvokeAsync(
            function: kickoffAndWaitFunction,
            arguments: new()
            {
                ["company"] = "CrewAI",
                ["topic"] = "Consumer AI Products"
            });

        Console.WriteLine(result);
    }
}
```