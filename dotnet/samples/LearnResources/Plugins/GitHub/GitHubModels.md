# Explanation

This file defines a set of C# classes used to represent data models for the GitHub REST API GET responses, as described in the official documentation (https://docs.github.com/en/rest). The classes include representations for a GitHub repository (`Repo`), user (`User`), issue (`Issue` and `IssueDetail`), and issue label (`Label`). Each property is decorated with the `JsonPropertyName` attribute to map JSON property names from the GitHub API to C# class properties for deserialization with `System.Text.Json`. These models are intended for use within the `Plugins` namespace and are marked as internal.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json.Serialization;

namespace Plugins;

/// <summary>
/// Models for GitHub REST API GET responses:
/// https://docs.github.com/en/rest
/// </summary>
internal static class GitHubModels
{
    public sealed class Repo
    {
        [JsonPropertyName("id")]
        public long Id { get; set; }

        [JsonPropertyName("full_name")]
        public string Name { get; set; }

        [JsonPropertyName("description")]
        public string Description { get; set; }

        [JsonPropertyName("html_url")]
        public string Url { get; set; }
    }

    public sealed class User
    {
        [JsonPropertyName("id")]
        public long Id { get; set; }

        [JsonPropertyName("login")]
        public string Login { get; set; }

        [JsonPropertyName("name")]
        public string Name { get; set; }

        [JsonPropertyName("company")]
        public string Company { get; set; }

        [JsonPropertyName("html_url")]
        public string Url { get; set; }
    }

    public class Issue
    {
        [JsonPropertyName("id")]
        public long Id { get; set; }

        [JsonPropertyName("number")]
        public int Number { get; set; }

        [JsonPropertyName("html_url")]
        public string Url { get; set; }

        [JsonPropertyName("title")]
        public string Title { get; set; }

        [JsonPropertyName("state")]
        public string State { get; set; }

        [JsonPropertyName("labels")]
        public Label[] Labels { get; set; }

        [JsonPropertyName("created_at")]
        public string WhenCreated { get; set; }

        [JsonPropertyName("closed_at")]
        public string WhenClosed { get; set; }
    }

    public sealed class IssueDetail : Issue
    {
        [JsonPropertyName("body")]
        public string Body { get; set; }
    }

    public sealed class Label
    {
        [JsonPropertyName("id")]
        public long Id { get; set; }

        [JsonPropertyName("name")]
        public string Name { get; set; }

        [JsonPropertyName("description")]
        public string Description { get; set; }
    }
}
```