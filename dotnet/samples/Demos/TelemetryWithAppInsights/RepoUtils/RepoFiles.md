# Explanation
This C# code defines a static class `RepoFiles` that provides a utility method to locate a folder named "prompt_template_samples" within the local repository directory structure. The method `SamplePluginsPath` searches upward from the location of the executing assembly for a directory named "prompt_template_samples". If found within 10 directory traversal attempts, it returns the full path to this folder. If not found, it throws a `DirectoryNotFoundException` with an explanatory message. This utility is intended for applications that require access to plugin samples organized in this folder.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.IO;
using System.Reflection;

internal static class RepoFiles
{
    /// <summary>
    /// Scan the local folders from the repo, looking for "prompt_template_samples" folder.
    /// </summary>
    /// <returns>The full path to prompt_template_samples</returns>
    public static string SamplePluginsPath()
    {
        const string Folder = "prompt_template_samples";

        static bool SearchPath(string pathToFind, out string result, int maxAttempts = 10)
        {
            var currDir = Path.GetFullPath(Assembly.GetExecutingAssembly().Location);
            bool found;
            do
            {
                result = Path.Join(currDir, pathToFind);
                found = Directory.Exists(result);
                currDir = Path.GetFullPath(Path.Combine(currDir, ".."));
            } while (maxAttempts-- > 0 && !found);

            return found;
        }

        if (!SearchPath(Folder, out var path))
        {
            throw new DirectoryNotFoundException("Plugins directory not found. The app needs the plugins from the repo to work.");
        }

        return path;
    }
}
```