# Explanation
This C# file defines the AzureAISearchConfig class, which is used to store configuration settings for connecting to an Azure AI Search service within the VectorStoreRAG application. The class is marked as internal and sealed, ensuring it can't be inherited or accessed outside its assembly. It contains two required properties: Endpoint, for the Azure AI Search service endpoint URL, and ApiKey, for the authentication key. The ConfigSectionName constant indicates the configuration section name in, for example, an appsettings.json file.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Azure AI Search service settings.
/// </summary>
internal sealed class AzureAISearchConfig
{
    public const string ConfigSectionName = "AzureAISearch";

    [Required]
    public string Endpoint { get; set; } = string.Empty;

    [Required]
    public string ApiKey { get; set; } = string.Empty;
}
```