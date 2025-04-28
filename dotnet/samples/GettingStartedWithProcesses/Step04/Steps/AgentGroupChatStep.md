# Explanation
This C# class `AgentGroupChatStep` is part of a semantic kernel-based agent framework. It defines a process step where a group chat involving multiple agents collaborates on a user's input. The class:
- Invokes the group chat service to process the user input.
- Emits the conversation messages as events during the chat.
- Collects the chat history.
- Summarizes the entire group chat conversation.
- Emits a completion event with the summary.

This facilitates a scenario where multiple agents contribute to a discussion, and their interactions are orchestrated and summarized for use in a larger AI process.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.ChatCompletion;

namespace Step04.Steps;

/// <summary>
/// This steps defines actions for the group chat in which to agents collaborate in
/// response to input from the primary agent.
/// </summary>
public class AgentGroupChatStep : KernelProcessStep
{
    public const string ChatServiceKey = $"{nameof(AgentGroupChatStep)}:{nameof(ChatServiceKey)}";
    public const string ReducerServiceKey = $"{nameof(AgentGroupChatStep)}:{nameof(ReducerServiceKey)}";

    public static class Functions
    {
        public const string InvokeAgentGroup = nameof(InvokeAgentGroup);
    }

    [KernelFunction(Functions.InvokeAgentGroup)]
    public async Task InvokeAgentGroupAsync(KernelProcessStepContext context, Kernel kernel, string input)
    {
        AgentGroupChat chat = kernel.GetRequiredService<AgentGroupChat>();

        // Reset chat state from previous invocation
        //await chat.ResetAsync();
        chat.IsComplete = false;

        ChatMessageContent message = new(AuthorRole.User, input);
        chat.AddChatMessage(message);
        await context.EmitEventAsync(new() { Id = AgentOrchestrationEvents.GroupMessage, Data = message });

        await foreach (ChatMessageContent response in chat.InvokeAsync())
        {
            await context.EmitEventAsync(new() { Id = AgentOrchestrationEvents.GroupMessage, Data = response });
        }

        ChatMessageContent[] history = await chat.GetChatMessagesAsync().Reverse().ToArrayAsync();

        // Summarize the group chat as a response to the primary agent
        string summary = await kernel.SummarizeHistoryAsync(ReducerServiceKey, history);

        await context.EmitEventAsync(new() { Id = AgentOrchestrationEvents.GroupCompleted, Data = summary });
    }
}
```