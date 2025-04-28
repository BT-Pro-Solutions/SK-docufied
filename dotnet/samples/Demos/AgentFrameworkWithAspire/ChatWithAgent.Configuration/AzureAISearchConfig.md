# Explanation
This C# code defines a configuration class `AzureAISearchConfig` used to store settings related to the Azure AI Search service. It includes constant string fields for the configuration section name and the connection string name, which helps centralize and standardize configuration keys used in the application.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace ChatWithAgent.Configuration;

/// <summary>
/// Azure AI Search service settings.
/// </summary>
public sealed class AzureAISearchConfig
{
    /// <summary>
    /// Configuration section name.
    /// </summary>
    public const string ConfigSectionName = "AzureAISearch";

    /// <summary>
    /// The name of the connection string of Azure AI Search service.
    /// </summary>
    public const string ConnectionStringName = "AzureAISearch";
}
```