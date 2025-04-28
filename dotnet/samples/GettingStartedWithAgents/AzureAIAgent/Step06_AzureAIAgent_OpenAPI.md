# Explanation

This code demonstrates how to invoke Open API functions (REST APIs defined by OpenAPI/Swagger) using the AzureAIAgent class from the Microsoft Semantic Kernel/Azure AI SDKs. The example does not leverage kernel function calling or use any custom kernel filters—instead, all function invocation is managed by the Azure AI Agent service. 

The specific scenario showcases how to:
- Load OpenAPI specifications (for countries and weather APIs) from embedded resources.
- Define an agent that is equipped with those Open API tools.
- Create a conversational thread for interaction.
- Send user prompts to the agent, which then calls the appropriate API tools to answer the queries, demonstrating tool chaining (e.g., using the result of one API call to inform the next).
- Clean up by deleting the thread and agent after use.

This example is meant for testing and learning purposes, integrating with Azure AI agent services in .NET.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Azure.AI.Projects;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents.AzureAI;
using Microsoft.SemanticKernel.ChatCompletion;
using Resources;

namespace GettingStarted.AzureAgents;

/// <summary>
/// This example demonstrates invoking Open API functions using <see cref="AzureAIAgent" />.
/// </summary>
/// <remarks>
/// Note: Open API invocation does not involve kernel function calling or kernel filters.
/// Azure Function invocation is managed entirely by the Azure AI Agent service.
/// </remarks>
public class Step06_AzureAIAgent_OpenAPI(ITestOutputHelper output) : BaseAzureAgentTest(output)
{
    [Fact]
    public async Task UseOpenAPIToolWithAgent()
    {
        // Retrieve Open API specifications
        string apiCountries = EmbeddedResource.Read("countries.json");
        string apiWeather = EmbeddedResource.Read("weather.json");

        // Define the agent
        Agent definition = await this.AgentsClient.CreateAgentAsync(
            TestConfiguration.AzureAI.ChatModelId,
            tools:
            [
                new OpenApiToolDefinition("RestCountries", "Retrieve country information", BinaryData.FromString(apiCountries), new OpenApiAnonymousAuthDetails()),
                new OpenApiToolDefinition("Weather", "Retrieve weather by location", BinaryData.FromString(apiWeather), new OpenApiAnonymousAuthDetails())
            ]);
        AzureAIAgent agent = new(definition, this.AgentsClient);

        // Create a thread for the agent conversation.
        Microsoft.SemanticKernel.Agents.AgentThread thread = new AzureAIAgentThread(this.AgentsClient, metadata: SampleMetadata);

        // Respond to user input
        try
        {
            await InvokeAgentAsync("What is the name and population of the country that uses currency with abbreviation THB");
            await InvokeAgentAsync("What is the weather in the capitol city of that country?");
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
        }

        // Local function to invoke agent and display the conversation messages.
        async Task InvokeAgentAsync(string input)
        {
            ChatMessageContent message = new(AuthorRole.User, input);
            this.WriteAgentChatMessage(message);

            await foreach (ChatMessageContent response in agent.InvokeAsync(message, thread))
            {
                this.WriteAgentChatMessage(response);
            }
        }
    }
}
```