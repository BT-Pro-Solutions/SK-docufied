# Explanation
This C# code defines an internal struct `EvaluationScoreType` used to represent different types of evaluation scores (such as BERT, BLEU, METEOR, and COMET) for configuring and running quality check examples. The struct holds an endpoint string and provides comparison operators, equality methods, and a custom hash code implementation, all while ensuring case-insensitive equality checks on the endpoint string.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Diagnostics.CodeAnalysis;

namespace QualityCheckWithFilters.Models;

/// <summary>
/// Internal representation of evaluation score type to configure and run examples.
/// </summary>
internal readonly struct EvaluationScoreType(string endpoint) : IEquatable<EvaluationScoreType>
{
    public string Endpoint { get; } = endpoint;

    public static EvaluationScoreType BERT = new("bert-score");
    public static EvaluationScoreType BLEU = new("bleu-score");
    public static EvaluationScoreType METEOR = new("meteor-score");
    public static EvaluationScoreType COMET = new("comet-score");

    public static bool operator ==(EvaluationScoreType left, EvaluationScoreType right) => left.Equals(right);
    public static bool operator !=(EvaluationScoreType left, EvaluationScoreType right) => !(left == right);

    /// <inheritdoc/>
    public override bool Equals([NotNullWhen(true)] object? obj) => obj is EvaluationScoreType other && this == other;

    /// <inheritdoc/>
    public bool Equals(EvaluationScoreType other) => string.Equals(this.Endpoint, other.Endpoint, StringComparison.OrdinalIgnoreCase);

    /// <inheritdoc/>
    public override int GetHashCode() => StringComparer.OrdinalIgnoreCase.GetHashCode(this.Endpoint ?? string.Empty);

    /// <inheritdoc/>
    public override string ToString() => this.Endpoint ?? string.Empty;
}
```