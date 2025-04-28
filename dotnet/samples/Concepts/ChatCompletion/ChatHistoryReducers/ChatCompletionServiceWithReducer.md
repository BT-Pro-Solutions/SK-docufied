# Explanation

This C# file defines the ChatCompletionServiceWithReducer class, an implementation of the IChatCompletionService interface. Its purpose is to preprocess a chat history using a provided IChatHistoryReducer before forwarding it to the underlying chat completion service. The reducer allows the chat history to be trimmed or altered (for instance, to reduce token count or remove unnecessary messages), ensuring that only the essential parts of the conversation are sent to the chat model. The class supports both batched (`GetChatMessageContentsAsync`) and streaming (`GetStreamingChatMessageContentsAsync`) message retrieval, and uses composition (wrapping an existing service and reducer) for flexible integration. This is useful in scenarios like prompt size management or adhering to token limits when working with large conversations and LLMs.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Runtime.CompilerServices;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

namespace ChatCompletion;

/// <summary>
/// Instance of <see cref="IChatCompletionService"/> which will invoke a delegate
/// to reduce the size of the <see cref="ChatHistory"/> before sending it to the model.
/// </summary>
public sealed class ChatCompletionServiceWithReducer(IChatCompletionService service, IChatHistoryReducer reducer) : IChatCompletionService
{
    private static IReadOnlyDictionary<string, object?> EmptyAttributes { get; } = new Dictionary<string, object?>();

    public IReadOnlyDictionary<string, object?> Attributes => EmptyAttributes;

    /// <inheritdoc/>
    public async Task<IReadOnlyList<ChatMessageContent>> GetChatMessageContentsAsync(
        ChatHistory chatHistory,
        PromptExecutionSettings? executionSettings = null,
        Kernel? kernel = null,
        CancellationToken cancellationToken = default)
    {
        var reducedMessages = await reducer.ReduceAsync(chatHistory, cancellationToken).ConfigureAwait(false);
        var reducedHistory = reducedMessages is null ? chatHistory : new ChatHistory(reducedMessages);

        return await service.GetChatMessageContentsAsync(reducedHistory, executionSettings, kernel, cancellationToken).ConfigureAwait(false);
    }

    /// <inheritdoc/>
    public async IAsyncEnumerable<StreamingChatMessageContent> GetStreamingChatMessageContentsAsync(
        ChatHistory chatHistory,
        PromptExecutionSettings? executionSettings = null,
        Kernel? kernel = null,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        var reducedMessages = await reducer.ReduceAsync(chatHistory, cancellationToken).ConfigureAwait(false);
        var history = reducedMessages is null ? chatHistory : new ChatHistory(reducedMessages);

        var messages = service.GetStreamingChatMessageContentsAsync(history, executionSettings, kernel, cancellationToken);
        await foreach (var message in messages)
        {
            yield return message;
        }
    }
}
```