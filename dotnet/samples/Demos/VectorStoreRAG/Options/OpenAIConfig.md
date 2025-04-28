# Explanation
This C# file defines an internal configuration class, OpenAIConfig, for storing settings required to connect to an OpenAI service. The configuration includes the model identifier (ModelId), the API key (ApiKey), and an optional organization ID (OrgId). The class uses data annotations to require the ModelId and ApiKey fields. The class is likely used within an application to bind and validate settings from a configuration section named "OpenAI".

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// OpenAI service settings.
/// </summary>
internal sealed class OpenAIConfig
{
    public const string ConfigSectionName = "OpenAI";

    [Required]
    public string ModelId { get; set; } = string.Empty;

    [Required]
    public string ApiKey { get; set; } = string.Empty;

    [Required]
    public string? OrgId { get; set; } = null;
}
```