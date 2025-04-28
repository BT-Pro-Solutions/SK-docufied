# Explanation

This C# file demonstrates how to create and interact with Azure OpenAI agents using the Microsoft Semantic Kernel framework, specifically focusing on integrating custom plugins into agents. The class `Step02_AzureAIAgent_Plugins` includes two test methods:

1. **UseAzureAgentWithPlugin**: This test creates an agent with a `MenuPlugin`, configures it to answer questions about a menu, and interacts with it using various messages, simulating a chat session.
2. **UseAzureAgentWithPluginEnumParameter**: This test creates an agent with a `WidgetFactory` plugin and sends a user prompt to invoke its capabilities.

The class provides helper methods for agent and thread management, as well as for displaying the responses. It also ensures proper cleanup of resources after tests. This code is useful for developers looking to extend Azure OpenAI agents with custom functionality via kernel plugins and interact with them programmatically.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.Agents.AzureAI;
using Microsoft.SemanticKernel.ChatCompletion;
using Plugins;

namespace GettingStarted.AzureAgents;

/// <summary>
/// Demonstrate creation of <see cref="AzureAIAgent"/> with a <see cref="KernelPlugin"/>,
/// and then eliciting its response to explicit user messages.
/// </summary>
public class Step02_AzureAIAgent_Plugins(ITestOutputHelper output) : BaseAzureAgentTest(output)
{
    [Fact]
    public async Task UseAzureAgentWithPlugin()
    {
        // Define the agent
        AzureAIAgent agent = await CreateAzureAgentAsync(
                plugin: KernelPluginFactory.CreateFromType<MenuPlugin>(),
                instructions: "Answer questions about the menu.",
                name: "Host");

        // Create a thread for the agent conversation.
        AgentThread thread = new AzureAIAgentThread(this.AgentsClient, metadata: SampleMetadata);

        // Respond to user input
        try
        {
            await InvokeAgentAsync(agent, thread, "Hello");
            await InvokeAgentAsync(agent, thread, "What is the special soup and its price?");
            await InvokeAgentAsync(agent, thread, "What is the special drink and its price?");
            await InvokeAgentAsync(agent, thread, "Thank you");
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
        }
    }

    [Fact]
    public async Task UseAzureAgentWithPluginEnumParameter()
    {
        // Define the agent
        AzureAIAgent agent = await CreateAzureAgentAsync(plugin: KernelPluginFactory.CreateFromType<WidgetFactory>());

        // Create a thread for the agent conversation.
        AgentThread thread = new AzureAIAgentThread(this.AgentsClient, metadata: SampleMetadata);

        // Respond to user input
        try
        {
            await InvokeAgentAsync(agent, thread, "Create a beautiful red colored widget for me.");
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
        }
    }

    private async Task<AzureAIAgent> CreateAzureAgentAsync(KernelPlugin plugin, string? instructions = null, string? name = null)
    {
        // Define the agent
        Azure.AI.Projects.Agent definition = await this.AgentsClient.CreateAgentAsync(
            TestConfiguration.AzureAI.ChatModelId,
            name,
            null,
            instructions);

        AzureAIAgent agent = new(definition, this.AgentsClient);

        // Add to the agent's Kernel
        if (plugin != null)
        {
            agent.Kernel.Plugins.Add(plugin);
        }

        return agent;
    }

    // Local function to invoke agent and display the conversation messages.
    private async Task InvokeAgentAsync(AzureAIAgent agent, AgentThread thread, string input)
    {
        ChatMessageContent message = new(AuthorRole.User, input);
        this.WriteAgentChatMessage(message);

        await foreach (ChatMessageContent response in agent.InvokeAsync(message, thread))
        {
            this.WriteAgentChatMessage(response);
        }
    }
}
```