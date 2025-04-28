# Explanation

This C# code sample demonstrates how to use the `AzureAIAgent` class from the Microsoft Semantic Kernel framework to interact with Azure AI agents in a programmatic way. The `Step01_AzureAIAgent` class defines a test method that:
- Loads a YAML prompt template resource and converts it into a prompt template configuration.
- Creates an agent definition on Azure with instructions, name, and description derived from the YAML file.
- Sets up an `AzureAIAgent` with initial arguments (e.g., for generating a story about a "Dog" of "3" length).
- Starts an agent thread for conversation history.
- Invokes the agent, first with default arguments and then with override arguments ("Cat" instead of "Dog"), and outputs the agent's responses.
- Cleans up by deleting the thread and agent after the test.

The code is structured as a unit test using [xUnit](https://xunit.net/) conventions and can be adapted to showcase the similarity between using the `AzureAIAgent` and other agent types available in the Semantic Kernel SDK.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.Agents.AzureAI;
using Resources;

namespace GettingStarted.AzureAgents;

/// <summary>
/// This example demonstrates similarity between using <see cref="AzureAIAgent"/>
/// and other agent types.
/// </summary>
public class Step01_AzureAIAgent(ITestOutputHelper output) : BaseAzureAgentTest(output)
{
    [Fact]
    public async Task UseTemplateForAzureAgent()
    {
        // Define the agent
        string generateStoryYaml = EmbeddedResource.Read("GenerateStory.yaml");
        PromptTemplateConfig templateConfig = KernelFunctionYaml.ToPromptTemplateConfig(generateStoryYaml);
        // Instructions, Name and Description properties defined via the PromptTemplateConfig.
        Azure.AI.Projects.Agent definition = await this.AgentsClient.CreateAgentAsync(TestConfiguration.AzureAI.ChatModelId, templateConfig.Name, templateConfig.Description, templateConfig.Template);
        AzureAIAgent agent = new(
            definition,
            this.AgentsClient,
            templateFactory: new KernelPromptTemplateFactory(),
            templateFormat: PromptTemplateConfig.SemanticKernelTemplateFormat)
        {
            Arguments = new()
            {
                { "topic", "Dog" },
                { "length", "3" }
            }
        };

        // Create a thread for the agent conversation.
        AgentThread thread = new AzureAIAgentThread(this.AgentsClient, metadata: SampleMetadata);

        try
        {
            // Invoke the agent with the default arguments.
            await InvokeAgentAsync();

            // Invoke the agent with the override arguments.
            await InvokeAgentAsync(
                new()
                {
                    { "topic", "Cat" },
                    { "length", "3" },
                });
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
        }

        // Local function to invoke agent and display the response.
        async Task InvokeAgentAsync(KernelArguments? arguments = null)
        {
            await foreach (ChatMessageContent response in agent.InvokeAsync(thread, new() { KernelArguments = arguments }))
            {
                WriteAgentChatMessage(response);
            }
        }
    }
}
```