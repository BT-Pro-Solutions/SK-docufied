# Explanation
This C# file contains extension methods for the `Content` class. Most notably, it provides a method to convert a `Content` object into a corresponding `KernelContent` object. The conversion depends on the `Type` property of the `Content` instance, handling text, image, and audio types differently by creating specific subclasses of `KernelContent`. If the content type is unexpected, an exception is thrown.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using Microsoft.SemanticKernel;
using ModelContextProtocol.Protocol.Types;

namespace MCPClient;

/// <summary>
/// Extension methods for the <see cref="Content"/> class.
/// </summary>
public static class ContentExtensions
{
    /// <summary>
    /// Converts a <see cref="Content"/> object to a <see cref="KernelContent"/> object.
    /// </summary>
    /// <param name="content">The <see cref="Content"/> object to convert.</param>
    /// <returns>The corresponding <see cref="KernelContent"/> object.</returns>
    public static KernelContent ToKernelContent(this Content content)
    {
        return content.Type switch
        {
            "text" => new TextContent(content.Text),
            "image" => new ImageContent(Convert.FromBase64String(content.Data!), content.MimeType),
            "audio" => new AudioContent(Convert.FromBase64String(content.Data!), content.MimeType),
            _ => throw new InvalidOperationException($"Unexpected message content type '{content.Type}'"),
        };
    }
}
```