# Explanation
This file defines the IDataLoader interface for the VectorStoreRAG namespace. The interface specifies a contract for loading data into a data store, with a focus on ingesting text from PDF files. The main method, LoadPdf, accepts parameters for the PDF file path, batch size for parallel embedding generation and upload, a delay between batches (to avoid throttling), and a cancellation token to allow cooperative cancellation of the loading task. This interface is intended to be implemented by concrete classes that handle the actual details of loading PDF data into a vector data store, such as for use with retrieval-augmented generation (RAG) workflows.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace VectorStoreRAG;

/// <summary>
/// Interface for loading data into a data store.
/// </summary>
internal interface IDataLoader
{
    /// <summary>
    /// Load the text from a PDF file into the data store.
    /// </summary>
    /// <param name="pdfPath">The pdf file to load.</param>
    /// <param name="batchSize">Maximum number of parallel threads to generate embeddings and upload records.</param>
    /// <param name="betweenBatchDelayInMs">The number of milliseconds to delay between batches to avoid throttling.</param>
    /// <param name="cancellationToken">The <see cref="CancellationToken"/> to monitor for cancellation requests.</param>
    /// <returns>An async task that completes when the loading is complete.</returns>
    Task LoadPdf(string pdfPath, int batchSize, int betweenBatchDelayInMs, CancellationToken cancellationToken);
}
```