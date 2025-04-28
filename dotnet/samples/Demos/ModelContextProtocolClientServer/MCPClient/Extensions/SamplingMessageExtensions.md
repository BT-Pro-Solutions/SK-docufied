# Explanation
This C# file defines extension methods for the `SamplingMessage` type within the `MCPClient` namespace. It provides functionality to convert one or more `SamplingMessage` instances into `ChatMessageContent` objects, which seem to be a different representation of message content likely used for chat or kernel processing. The file includes two methods: one to convert a collection of `SamplingMessage` instances and another to convert a single `SamplingMessage`.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Collections.Generic;
using System.Linq;
using Microsoft.SemanticKernel;
using ModelContextProtocol.Protocol.Types;

namespace MCPClient;

/// <summary>
/// Extension methods for <see cref="SamplingMessage"/>.
/// </summary>
public static class SamplingMessageExtensions
{
    /// <summary>
    /// Converts a collection of <see cref="SamplingMessage"/> to a list of <see cref="ChatMessageContent"/>.
    /// </summary>
    /// <param name="samplingMessages">The collection of <see cref="SamplingMessage"/> to convert.</param>
    /// <returns>The corresponding list of <see cref="ChatMessageContent"/>.</returns>
    public static List<ChatMessageContent> ToChatMessageContents(this IEnumerable<SamplingMessage> samplingMessages)
    {
        return [.. samplingMessages.Select(ToChatMessageContent)];
    }

    /// <summary>
    /// Converts a <see cref="SamplingMessage"/> to a <see cref="ChatMessageContent"/>.
    /// </summary>
    /// <param name="message">The <see cref="SamplingMessage"/> to convert.</param>
    /// <returns>The corresponding <see cref="ChatMessageContent"/>.</returns>
    public static ChatMessageContent ToChatMessageContent(this SamplingMessage message)
    {
        return new ChatMessageContent(role: message.Role.ToAuthorRole(), items: [message.Content.ToKernelContent()]);
    }
}
```