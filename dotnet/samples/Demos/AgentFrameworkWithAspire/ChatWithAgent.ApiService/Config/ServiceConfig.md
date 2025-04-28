# Explanation
This C# file defines a `ServiceConfig` class used in the ChatWithAgent application to manage service-related configuration settings. It encapsulates a `HostConfig` instance which is initialized using a `ConfigurationManager` object passed to the constructor. This setup allows the service to access host-specific configuration details in a structured way.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using ChatWithAgent.Configuration;
using Microsoft.Extensions.Configuration;

namespace ChatWithAgent.ApiService.Config;

/// <summary>
/// Service configuration.
/// </summary>
public sealed class ServiceConfig
{
    private readonly HostConfig _hostConfig;

    /// <summary>
    /// Initializes a new instance of the <see cref="ServiceConfig"/> class.
    /// </summary>
    /// <param name="configurationManager">The configuration manager.</param>
    public ServiceConfig(ConfigurationManager configurationManager)
    {
        this._hostConfig = new HostConfig(configurationManager);
    }

    /// <summary>
    /// Host configuration.
    /// </summary>
    public HostConfig Host => this._hostConfig;
}
```