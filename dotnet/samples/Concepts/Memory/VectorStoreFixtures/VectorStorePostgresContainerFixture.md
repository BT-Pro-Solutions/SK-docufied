# Explanation
This C# file defines a test fixture class, `VectorStorePostgresContainerFixture`, used when running integration tests that require a Postgres database in a Docker container. It provides methods to initialize and dispose of the Postgres container, ensuring that the database is started before tests run and properly removed afterward. During initialization, it connects to Docker, launches the Postgres container, waits for it to be ready by attempting to open a connection, and ensures the necessary database extension (`vector`) is created if it doesn't already exist. The fixture also implements cleanup logic to remove the container after use.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Docker.DotNet;
using Npgsql;

namespace Memory.VectorStoreFixtures;

/// <summary>
/// Fixture to use for creating a Postgres container before tests and delete it after tests.
/// </summary>
public class VectorStorePostgresContainerFixture : IAsyncLifetime
{
    private DockerClient? _dockerClient;
    private string? _postgresContainerId;

    public async Task InitializeAsync()
    {
    }

    public async Task ManualInitializeAsync()
    {
        if (this._postgresContainerId == null)
        {
            // Connect to docker and start the docker container.
            using var dockerClientConfiguration = new DockerClientConfiguration();
            this._dockerClient = dockerClientConfiguration.CreateClient();
            this._postgresContainerId = await VectorStoreInfra.SetupPostgresContainerAsync(this._dockerClient);

            // Delay until the Postgres server is ready.
            var connectionString = TestConfiguration.Postgres.ConnectionString;
            var succeeded = false;
            var attemptCount = 0;
            while (!succeeded && attemptCount++ < 10)
            {
                try
                {
                    NpgsqlDataSourceBuilder dataSourceBuilder = new(connectionString);
                    dataSourceBuilder.UseVector();
                    using var dataSource = dataSourceBuilder.Build();
                    NpgsqlConnection connection = await dataSource.OpenConnectionAsync().ConfigureAwait(false);

                    await using (connection)
                    {
                        // Create extension vector if it doesn't exist
                        await using (NpgsqlCommand command = new("CREATE EXTENSION IF NOT EXISTS vector", connection))
                        {
                            await command.ExecuteNonQueryAsync();
                        }
                    }
                }
                catch (Exception)
                {
                    await Task.Delay(1000);
                }
            }
        }
    }

    public async Task DisposeAsync()
    {
        if (this._dockerClient != null && this._postgresContainerId != null)
        {
            // Delete docker container.
            await VectorStoreInfra.DeleteContainerAsync(this._dockerClient, this._postgresContainerId);
        }
    }
}
```