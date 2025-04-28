# Explanation
This code defines an OpenAIOptions class, which is intended for use in a .NET application to represent configuration settings required to connect to the OpenAI chat completion service. The class includes two required properties: ChatModelId (for specifying the chat model to use) and ApiKey (for authenticating requests). The SectionName constant ("OpenAI") can be used to map configuration sections. Data annotations ensure that the properties are mandatory when used with configuration binding or validation mechanisms.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace StepwisePlannerMigration.Options;

/// <summary>
/// Configuration for OpenAI chat completion service.
/// </summary>
public class OpenAIOptions
{
    public const string SectionName = "OpenAI";

    [Required]
    public string ChatModelId { get; set; }

    [Required]
    public string ApiKey { get; set; }
}
```
