# Explanation
The `IPlanProvider` interface defines a contract for retrieving a previously generated chat "plan" (of type `ChatHistory`) given a filename. This is intended for demonstration purposes and suggests that implementations would retrieve saved plans from files. The interface is part of the `StepwisePlannerMigration.Services` namespace, and it relies on types from the `Microsoft.SemanticKernel.ChatCompletion` namespace.

```csharp
// Copyright (c) Microsoft. All rights reserved.

#pragma warning disable IDE0005 // Using directive is unnecessary

using Microsoft.SemanticKernel.ChatCompletion;

#pragma warning restore IDE0005 // Using directive is unnecessary

namespace StepwisePlannerMigration.Services;

/// <summary>
/// Interface to get a previously generated plan from file for demonstration purposes.
/// </summary>
public interface IPlanProvider
{
    ChatHistory GetPlan(string fileName);
}
```