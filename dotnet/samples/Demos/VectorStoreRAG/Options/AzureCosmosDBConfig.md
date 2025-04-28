# Explanation

This code defines the `AzureCosmosDBConfig` class, which encapsulates configuration settings for connecting to Azure CosmosDB services in two different modes: MongoDB API and NoSQL API. The class uses data annotations to specify that both the connection string and the database name are required. It also provides constant strings to identify the configuration section names for each CosmosDB API variant. This class is intended for internal use within the `VectorStoreRAG.Options` namespace.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace VectorStoreRAG.Options;

/// <summary>
/// Azure CosmosDB service settings for use with AzureCosmosDBMongoDB and AzureCosmosDBNoSQL.
/// </summary>
internal sealed class AzureCosmosDBConfig
{
    public const string MongoDBConfigSectionName = "AzureCosmosDBMongoDB";
    public const string NoSQLConfigSectionName = "AzureCosmosDBNoSQL";

    [Required]
    public string ConnectionString { get; set; } = string.Empty;

    [Required]
    public string DatabaseName { get; set; } = string.Empty;
}
```