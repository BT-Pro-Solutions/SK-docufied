# Explanation

This code file demonstrates how to perform Retrieval-Augmented Generation (RAG) using Bing's text search capabilities integrated with Microsoft Semantic Kernel. It provides several unit test-like examples showing how to:

- Use Bing's text search as a grounding information source for prompts when using the Semantic Kernel and OpenAI chat completion models.
- Incorporate search results into prompts, including using Handlebars templates to structure and cite retrieved information.
- Filter searches to specific sites (e.g., Microsoft Developer Blogs).
- Present search results with additional metadata such as snippets and crawl dates for richer citation.

Each example shows how to wrap Bing text search functionality into a Semantic Kernel plugin and utilize it within a prompt pipeline, supporting the construction of more contextually-grounded and referenceable responses.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Data;
using Microsoft.SemanticKernel.Plugins.Web.Bing;
using Microsoft.SemanticKernel.PromptTemplates.Handlebars;

namespace RAG;

/// <summary>
/// This example shows how to perform RAG with an <see cref="ITextSearch"/>.
/// </summary>
public sealed class Bing_RagWithTextSearch(ITestOutputHelper output) : BaseTest(output)
{
    /// <summary>
    /// Show how to create a default <see cref="KernelPlugin"/> from an <see cref="ITextSearch"/> and use it to
    /// add grounding context to a prompt.
    /// </summary>
    [Fact]
    public async Task RagWithBingTextSearchAsync()
    {
        // Create a kernel with OpenAI chat completion
        IKernelBuilder kernelBuilder = Kernel.CreateBuilder();
        kernelBuilder.AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey);
        Kernel kernel = kernelBuilder.Build();

        // Create a text search using Bing search
        var textSearch = new BingTextSearch(new(TestConfiguration.Bing.ApiKey));

        // Build a text search plugin with Bing search and add to the kernel
        var searchPlugin = textSearch.CreateWithSearch("SearchPlugin");
        kernel.Plugins.Add(searchPlugin);

        // Invoke prompt and use text search plugin to provide grounding information
        var query = "What is the Semantic Kernel?";
        KernelArguments arguments = new() { { "query", query } };
        Console.WriteLine(await kernel.InvokePromptAsync("{{SearchPlugin.Search $query}}. {{$query}}", arguments));
    }

    /// <summary>
    /// Show how to create a default <see cref="KernelPlugin"/> from an <see cref="ITextSearch"/> and use it to
    /// add grounding context to a Handlebars prompt and include citations in the response.
    /// </summary>
    [Fact]
    public async Task RagWithBingTextSearchIncludingCitationsAsync()
    {
        // Create a kernel with OpenAI chat completion
        IKernelBuilder kernelBuilder = Kernel.CreateBuilder();
        kernelBuilder.AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey);
        Kernel kernel = kernelBuilder.Build();

        // Create a text search using Bing search
        var textSearch = new BingTextSearch(new(TestConfiguration.Bing.ApiKey));

        // Build a text search plugin with Bing search and add to the kernel
        var searchPlugin = textSearch.CreateWithGetTextSearchResults("SearchPlugin");
        kernel.Plugins.Add(searchPlugin);

        // Invoke prompt and use text search plugin to provide grounding information
        var query = "What is the Semantic Kernel?";
        string promptTemplate = """
            {{#with (SearchPlugin-GetTextSearchResults query)}}  
              {{#each this}}  
                Name: {{Name}}
                Value: {{Value}}
                Link: {{Link}}
                -----------------
              {{/each}}  
            {{/with}}  

            {{query}}

            Include citations to the relevant information where it is referenced in the response.
            """;
        KernelArguments arguments = new() { { "query", query } };
        HandlebarsPromptTemplateFactory promptTemplateFactory = new();
        Console.WriteLine(await kernel.InvokePromptAsync(
            promptTemplate,
            arguments,
            templateFormat: HandlebarsPromptTemplateFactory.HandlebarsTemplateFormat,
            promptTemplateFactory: promptTemplateFactory
        ));
    }

    /// <summary>
    /// Show how to create a default <see cref="KernelPlugin"/> from an <see cref="ITextSearch"/> and use it to
    /// add grounding context to a Handlebars prompt and include citations in the response.
    /// </summary>
    [Fact]
    public async Task RagWithBingTextSearchIncludingTimeStampedCitationsAsync()
    {
        // Create a kernel with OpenAI chat completion
        IKernelBuilder kernelBuilder = Kernel.CreateBuilder();
        kernelBuilder.AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey);
        Kernel kernel = kernelBuilder.Build();

        // Create a text search using Bing search
        var textSearch = new BingTextSearch(new(TestConfiguration.Bing.ApiKey));

        // Build a text search plugin with Bing search and add to the kernel
        var searchPlugin = textSearch.CreateWithGetSearchResults("SearchPlugin");
        kernel.Plugins.Add(searchPlugin);

        // Invoke prompt and use text search plugin to provide grounding information
        var query = "What is the Semantic Kernel?";
        string promptTemplate = """
            {{#with (SearchPlugin-GetSearchResults query)}}  
              {{#each this}}  
                Name: {{Name}}
                Snippet: {{Snippet}}
                Link: {{DisplayUrl}}
                Date Last Crawled: {{DateLastCrawled}}
                -----------------
              {{/each}}  
            {{/with}}  

            {{query}}

            Include citations to and the date of the relevant information where it is referenced in the response.
            """;
        KernelArguments arguments = new() { { "query", query } };
        HandlebarsPromptTemplateFactory promptTemplateFactory = new();
        Console.WriteLine(await kernel.InvokePromptAsync(
            promptTemplate,
            arguments,
            templateFormat: HandlebarsPromptTemplateFactory.HandlebarsTemplateFormat,
            promptTemplateFactory: promptTemplateFactory
        ));
    }

    /// <summary>
    /// Show how to create a default <see cref="KernelPlugin"/> from an <see cref="ITextSearch"/> and use it to
    /// add grounding context to a Handlebars prompt that include full web pages.
    /// </summary>
    [Fact]
    public async Task RagWithBingTextSearchUsingDevBlogsSiteAsync()
    {
        // Create a kernel with OpenAI chat completion
        IKernelBuilder kernelBuilder = Kernel.CreateBuilder();
        kernelBuilder.AddOpenAIChatCompletion(
                modelId: TestConfiguration.OpenAI.ChatModelId,
                apiKey: TestConfiguration.OpenAI.ApiKey);
        Kernel kernel = kernelBuilder.Build();

        // Create a text search using Bing search
        var textSearch = new BingTextSearch(new(TestConfiguration.Bing.ApiKey));

        // Build a text search plugin with Bing search and add to the kernel
        var filter = new TextSearchFilter().Equality("site", "devblogs.microsoft.com");
        var searchOptions = new TextSearchOptions() { Filter = filter };
        var searchPlugin = KernelPluginFactory.CreateFromFunctions(
            "SearchPlugin", "Search Microsoft Developer Blogs site only",
            [textSearch.CreateGetTextSearchResults(searchOptions: searchOptions)]);
        kernel.Plugins.Add(searchPlugin);

        // Invoke prompt and use text search plugin to provide grounding information
        var query = "What is the Semantic Kernel?";
        string promptTemplate = """
            {{#with (SearchPlugin-GetTextSearchResults query)}}  
              {{#each this}}  
                Name: {{Name}}
                Value: {{Value}}
                Link: {{Link}}
                -----------------
              {{/each}}  
            {{/with}}  

            {{query}}

            Include citations to the relevant information where it is referenced in the response.
            """;
        KernelArguments arguments = new() { { "query", query } };
        HandlebarsPromptTemplateFactory promptTemplateFactory = new();
        Console.WriteLine(await kernel.InvokePromptAsync(
            promptTemplate,
            arguments,
            templateFormat: HandlebarsPromptTemplateFactory.HandlebarsTemplateFormat,
            promptTemplateFactory: promptTemplateFactory
        ));
    }
}
```