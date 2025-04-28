# Explanation
This C# code defines a static class `ProcessEvents` within the `ProcessFramework.Aspire.ProcessOrchestrator.Models` namespace. The class contains readonly string fields that represent event names used in a process orchestrator system, such as translating and summarizing documents. Each string field is assigned a value equal to its own name using the `nameof` operator, ensuring consistency and avoiding hard-coded string literals.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace ProcessFramework.Aspire.ProcessOrchestrator.Models;

public static class ProcessEvents
{
    public static readonly string TranslateDocument = nameof(TranslateDocument);
    public static readonly string DocumentTranslated = nameof(DocumentTranslated);
    public static readonly string SummarizeDocument = nameof(SummarizeDocument);
    public static readonly string DocumentSummarized = nameof(DocumentSummarized);
}
```