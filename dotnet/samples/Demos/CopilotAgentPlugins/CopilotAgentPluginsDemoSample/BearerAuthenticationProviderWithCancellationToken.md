# Explanation

This C# class, `BearerAuthenticationProviderWithCancellationToken`, is designed to handle bearer authentication for HTTP requests using Microsoft identity services. It retrieves an access token from Microsoft Identity via the MSAL (Microsoft Authentication Library) public client, reading configuration values (ClientId, TenantId) from an `IConfiguration` instance—typically sourced from an appsettings.json file.

The class provides a method, `AuthenticateRequestAsync`, which attaches a bearer token to an `HttpRequestMessage`. The token is obtained using the silent flow, or, if unavailable, via device code authentication, and supports operation cancellation through a `CancellationToken`.

This is commonly used for applications interfacing with Microsoft Graph APIs or other Azure-protected resources, where secure authentication and token acquisition are required in a cancellable and reusable way.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Net.Http.Headers;
using Microsoft.Extensions.Configuration;
using Microsoft.Identity.Client;

/// <summary>
/// Retrieves a token via the provided delegate and applies it to HTTP requests using the
/// "bearer" authentication scheme.
/// </summary>
public class BearerAuthenticationProviderWithCancellationToken
{
    private readonly IPublicClientApplication _client;

    /// <summary>
    /// Creates an instance of the <see cref="BearerAuthenticationProviderWithCancellationToken"/> class.
    /// </summary>
    /// <param name="configuration">The configuration instance to read settings from.</param>
    public BearerAuthenticationProviderWithCancellationToken(IConfiguration configuration)
    {
        ArgumentNullException.ThrowIfNull(configuration);
        var clientId = configuration["MSGraph:ClientId"];
        var tenantId = configuration["MSGraph:TenantId"];

        if (string.IsNullOrEmpty(clientId) || string.IsNullOrEmpty(tenantId))
        {
            throw new InvalidOperationException("Please provide valid MSGraph configuration in appsettings.Development.json file.");
        }

        this._client = PublicClientApplicationBuilder
            .Create(clientId)
            .WithAuthority($"https://login.microsoftonline.com/{tenantId}")
            .WithDefaultRedirectUri()
            .Build();
    }

    /// <summary>
    /// Applies the token to the provided HTTP request message.
    /// </summary>
    /// <param name="request">The HTTP request message.</param>
    /// <param name="cancellationToken"></param>
    public async Task AuthenticateRequestAsync(HttpRequestMessage request, CancellationToken cancellationToken = default)
    {
        var token = await this.GetAccessTokenAsync(cancellationToken).ConfigureAwait(false);
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);
    }
    private async Task<string> GetAccessTokenAsync(CancellationToken cancellationToken)
    {
        var scopes = new string[] { "https://graph.microsoft.com/.default" };
        try
        {
            var authResult = await this._client.AcquireTokenSilent(scopes, (await this._client.GetAccountsAsync().ConfigureAwait(false)).FirstOrDefault()).ExecuteAsync(cancellationToken).ConfigureAwait(false);
            return authResult.AccessToken;
        }
        catch
        {
            var authResult = await this._client.AcquireTokenWithDeviceCode(scopes, deviceCodeResult =>
            {
                Console.WriteLine(deviceCodeResult.Message);
                return Task.CompletedTask;
            }).ExecuteAsync(cancellationToken).ConfigureAwait(false);
            return authResult.AccessToken;
        }
    }
}
```
