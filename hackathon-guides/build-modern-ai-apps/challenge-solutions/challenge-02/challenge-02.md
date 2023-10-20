# Solution for Challenge 02

1. `AzureCognitiveSearchVectorMemory.cs`

The field containing the vector must be searchable, so `IsSearchable` should be set to `true`. For more details, see [Searchable content in Cognitive Search](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview#searchable-content-in-cognitive-search).

The number of dimensions of the vector is specified by `ModelDimensions`. This number should be aligned with the number of dimensions of the Azure Open AI embedding model (which is currently 1536). For more details, see [Tutorial: Explore Azure OpenAI Service embeddings and document search](https://learn.microsoft.com/en-us/azure/ai-services/openai/tutorials/embeddings?tabs=command-line).

Algorithm configurations are implemented in the `VectorSearchAlgorithmConfiguration` class. Note the specific algorithm that we are using - Hierarchical Navigable Small World (HNSW). For more details, see [Relevance and ranking in vector search](https://learn.microsoft.com/en-us/azure/search/vector-search-ranking).

2. `CosmosDbService.cs`

The list of monitored containers is provided in configuration and is mapped to `_settings.MonitoredContainers`.

Use `monitoredContainerName` as the name of the iteration variable.

Following the instructions in the note, also add `.WithStartTime(DateTime.MinValue.ToUniversalTime())`.