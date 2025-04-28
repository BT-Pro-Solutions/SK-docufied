# Explanation
This C# class, `DocumentInfo`, defines a simple data container for storing information about a generated document. It includes three publicly accessible and serializable properties: `Id`, `Title`, and `Content`, each initialized to an empty string. The class is part of the `ProcessWithCloudEvents.Processes.Models` namespace and is designed to be used as a parameter and state type across multiple process steps, ensuring compatibility with serialization requirements.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace ProcessWithCloudEvents.Processes.Models;

/// <summary>
/// Object used to store generated document data
/// Since this object is used as parameter and state type by multiple steps,
/// Its members must be public and serializable
/// </summary>
//[DataContract]
public class DocumentInfo
{
    /// <summary>
    /// Id of the document
    /// </summary>
    public string Id { get; set; } = string.Empty;
    /// <summary>
    /// Title of the document
    /// </summary>
    public string Title { get; set; } = string.Empty;
    /// <summary>
    /// Content of the document
    /// </summary>
    public string Content { get; set; } = string.Empty;
}
```