# Explanation
This code defines a configuration class for connecting to the Azure AI Content Safety service. The AzureContentSafetyOptions class is designed to be used with .NET's configuration and dependency injection system. It contains two properties, Endpoint and ApiKey, both of which are required and will hold the Azure Content Safety service endpoint URL and the associated API key, respectively. The SectionName constant indicates the configuration section from which these values should be retrieved ("AzureContentSafety").

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ContentSafety.Options;

/// <summary>
/// Configuration for Azure AI Content Safety service.
/// </summary>
public class AzureContentSafetyOptions
{
    public const string SectionName = "AzureContentSafety";

    [Required]
    public string Endpoint { get; set; }

    [Required]
    public string ApiKey { get; set; }
}
```