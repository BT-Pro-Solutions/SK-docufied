# Explanation
This C# code defines a static class `EmbeddedResource` that provides a method to read text from embedded resource files within the same assembly. It uses reflection to locate the assembly and then access the resource based on the namespace and the resource file name. The contents of the resource are returned as a string.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using System.IO;
using System.Reflection;

namespace ChatWithAgent.ApiService.Resources;

/// <summary>
/// Reads embedded resources from the assembly.
/// </summary>
public static class EmbeddedResource
{
    private static readonly string? s_namespace = typeof(EmbeddedResource).Namespace;

    internal static string Read(string fileName)
    {
        // Get the current assembly. Note: this class is in the same assembly where the embedded resources are stored.
        Assembly assembly =
            typeof(EmbeddedResource).GetTypeInfo().Assembly ??
            throw new InvalidOperationException($"[{s_namespace}] {fileName} assembly not found");

        // Resources are mapped like types, using the namespace and appending "." (dot) and the file name
        var resourceName = $"{s_namespace}." + fileName;
        using Stream resource =
            assembly.GetManifestResourceStream(resourceName) ??
            throw new InvalidOperationException($"{resourceName} resource not found");

        // Return the resource content, in text format.
        using var reader = new StreamReader(resource);

        return reader.ReadToEnd();
    }
}
```