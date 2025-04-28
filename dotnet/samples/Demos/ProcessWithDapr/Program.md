# Explanation
This code file defines the entry point for an ASP.NET Core web application that integrates Microsoft Semantic Kernel and Dapr for actor-based processing. Key points:
- It configures logging to output to the console and debugger.
- The Semantic Kernel is registered with dependency injection, supporting advanced functionality such as processes.
- Dapr actors are configured, with specific actors for process management added.
- Controllers are registered, enabling standard HTTP endpoints.
- The application pipeline is set up, switching between developer diagnostics and production configurations like HTTPS redirection and authorization.
- Finally, HTTP endpoints and Dapr actor handlers are mapped, and the application is started.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

var builder = WebApplication.CreateBuilder(args);

// Configure logging
builder.Services.AddLogging((logging) =>
{
    logging.AddConsole();
    logging.AddDebug();
});

// Configure the Kernel with DI. This is required for dependency injection to work with processes.
builder.Services.AddKernel();

// Configure Dapr
builder.Services.AddActors(static options =>
{
    // Register the actors required to run Processes
    options.AddProcessActors();
});

builder.Services.AddControllers();
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Configure the HTTP request pipeline.
    app.UseHttpsRedirection();
    app.UseAuthorization();
}

app.MapControllers();
app.MapActorsHandlers();
app.Run();
```