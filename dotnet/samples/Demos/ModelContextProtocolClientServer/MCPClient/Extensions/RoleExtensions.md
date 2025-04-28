# Explanation
This C# file defines an extension class `RoleExtensions` that provides a method to convert a `Role` enum value into a corresponding `AuthorRole` enum value. The conversion is implemented using a switch expression, handling specific known roles (`User` and `Assistant`) and throwing an exception for any unexpected role values. This helps in mapping roles used in the MCP (ModelContextProtocol) to roles understood by the Semantic Kernel chat completion API.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using Microsoft.SemanticKernel.ChatCompletion;
using ModelContextProtocol.Protocol.Types;

namespace MCPClient;

/// <summary>
/// Extension methods for the <see cref="Role"/> enum.
/// </summary>
internal static class RoleExtensions
{
    /// <summary>
    /// Converts a <see cref="Role"/> to a <see cref="AuthorRole"/>.
    /// </summary>
    /// <param name="role">The MCP role to convert.</param>
    /// <returns>The corresponding <see cref="AuthorRole"/>.</returns>
    public static AuthorRole ToAuthorRole(this Role role)
    {
        return role switch
        {
            Role.User => AuthorRole.User,
            Role.Assistant => AuthorRole.Assistant,
            _ => throw new InvalidOperationException($"Unexpected role '{role}'")
        };
    }
}
```