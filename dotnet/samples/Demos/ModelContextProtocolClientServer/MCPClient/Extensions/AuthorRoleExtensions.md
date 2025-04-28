# Explanation
This C# file defines extension methods for converting an `AuthorRole` enumeration value to a corresponding `Role` enumeration value used within the ModelContextProtocol framework. The primary method `ToMCPRole` converts `AuthorRole.User` to `Role.User` and `AuthorRole.Assistant` to `Role.Assistant`. If an unexpected role is passed, it throws an `InvalidOperationException`. This facilitates role translation between different parts of the system.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using Microsoft.SemanticKernel.ChatCompletion;
using ModelContextProtocol.Protocol.Types;

namespace MCPClient;

/// <summary>
/// Extension methods for the <see cref="AuthorRole"/>.
/// </summary>
internal static class AuthorRoleExtensions
{
    /// <summary>
    /// Converts a <see cref="AuthorRole"/> to a <see cref="Role"/>.
    /// </summary>
    /// <param name="role">The author role to convert.</param>
    /// <returns>The corresponding <see cref="Role"/>.</returns>
    public static Role ToMCPRole(this AuthorRole role)
    {
        if (role == AuthorRole.User)
        {
            return Role.User;
        }

        if (role == AuthorRole.Assistant)
        {
            return Role.Assistant;
        }

        throw new InvalidOperationException($"Unexpected role '{role}'");
    }
}
```