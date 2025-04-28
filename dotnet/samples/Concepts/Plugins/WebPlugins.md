# Explanation
This C# code defines a sample plugin class, `WebPlugins`, demonstrating how to use the Semantic Kernel web plugins to download a file (specifically, the Semantic Kernel logo PNG) from a remote URL to a temporary directory on the local machine. The `DownloadSKLogoAsync` test method creates a temporary folder, configures the `WebFileDownloadPlugin` with domain and folder restrictions, and downloads the file to the specified path. It ensures that the download path is allowed and cleans up by deleting the temporary directory afterward. The class is intended for testing and demonstration purposes, assuming dependencies such as `BaseTest`, `ITestOutputHelper`, and the Semantic Kernel libraries are available.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel.Plugins.Web;

namespace Plugins;

/// <summary>
/// Sample showing how to use the Semantic Kernel web plugins correctly.
/// </summary>
public sealed class WebPlugins(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Shows how to download to a temporary directory on the local machine.
    /// </summary>
    [Fact]
    public async Task DownloadSKLogoAsync()
    {
        var uri = new Uri("https://raw.githubusercontent.com/microsoft/semantic-kernel/refs/heads/main/docs/images/sk_logo.png");
        var folderPath = Path.Combine(Path.GetTempPath(), Guid.NewGuid().ToString());
        var filePath = Path.Combine(folderPath, "sk_logo.png");

        try
        {
            Directory.CreateDirectory(folderPath);

            var webFileDownload = new WebFileDownloadPlugin(this.LoggerFactory)
            {
                AllowedDomains = ["raw.githubusercontent.com"],
                AllowedFolders = [folderPath]
            };

            await webFileDownload.DownloadToFileAsync(uri, filePath);

            if (Path.Exists(filePath))
            {
                Output.WriteLine($"Successfully downloaded to {filePath}");
            }
        }
        finally
        {
            if (Path.Exists(folderPath))
            {
                Directory.Delete(folderPath, true);
            }
        }
    }
}
```