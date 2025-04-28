# Explanation

This C# code defines a utility class `JsonSchemaGenerator` for generating a JSON schema string representation of a given .NET type. It utilizes features from the `Microsoft.SemanticKernel` and `Microsoft.Extensions.AI` libraries to build the schema. The method `FromType<TSchemaType>()` takes a type parameter, configures serialization and schema options (such as disallowing unmapped members and additional properties), and returns the generated JSON schema as a string. This class is useful for scenarios where standardized JSON schema representations are needed, such as validating API contracts or integrating with AI systems that expect strict input/output validation.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using System.Text.Json;
using System.Text.Json.Serialization;
using Microsoft.Extensions.AI;
using Microsoft.SemanticKernel;

namespace Step04;

internal static class JsonSchemaGenerator
{
    /// <summary>
    /// Wrapper for generating a JSON schema as string from a .NET type.
    /// </summary>
    public static string FromType<TSchemaType>()
    {
        JsonSerializerOptions options = new(JsonSerializerOptions.Default)
        {
            UnmappedMemberHandling = JsonUnmappedMemberHandling.Disallow,
        };
        AIJsonSchemaCreateOptions config = new()
        {
            IncludeSchemaKeyword = false,
            DisallowAdditionalProperties = true,
        };

        return KernelJsonSchemaBuilder.Build(typeof(TSchemaType), "Intent Result", config).AsJson();
    }
}
```
