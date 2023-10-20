# Solution for Challenge 03

1. `ChatBuilder.cs`

The `WithMemories` method populates the `_memories` private property in `ChatBuilder`. It uses `EmbeddingUtility.Transform` to get the text representation of the entities in the memories. The `Transform` method uses the `ModelRegistry` to determine the type of the entity and the `EmbeddingField` attribute to determine the prefix used when serializing the property value to text. The `ModelRegistry` is also used to determine the name of the entity, using the `NamingProperties` attribute. For more details, see [Notes on the solution](#notes-on-the-solution).

`memoriesPrompt` will be a simple concatenation of the JSON representations of the objects from `_memories`.

The `AddSystemMessage` method adds a system message to the chat. The message to be added is the system prompt stored in `_systemPrompt` when `WithSystemPrompt` is called.

The `AddMesage` method adds the user message to the chat. The messages are stored in `_messages` when `WithMessageHistory` is called. The `AuthorRole` property on each message is defined by Semantic Kernel and can be one of `system`, `assistant`, or `user`.

2. `ChatService.cs`

The prompt message shoud contain the sender name (`user`), the number of user prompt tokens (`result.UserPromptTokens`), the user prompt and the embedding of the user prompt.

The completion message should contain the sender name (`assistant`), the number of response tokens (`result.ResponseTokens`), and the text of completion.

3. `SemanticKernelRAGService.cs`

The `WithSystemPrompt` method sets the `_systemPrompt` private property in `ChatBuilder`.

The `WithMemories` method sets the `_memories` private property in `ChatBuilder`.

The `WithMessageHistory` method sets the `_messages` private property in `ChatBuilder`.

A chat completion Semantic Kernel flow returns one or more completions. For the purpose of this exercise we are only interested in the first completion, returned by `completionResults[0]`.

When returning results:

- The completion is available in `reply.Content`.
- The user prompt is the first message in the chat history.
- The tokens consumptions are available in `rawResult.Usage`.
- The user prompt embedding is available in `userPromptEmbedding` (retrieved earlier as the last embedding returned by the `TextEmbeddingObjectMemorySkill`).

## Notes on the solution

The basic idea of Model Registry is to have a centralized way for managing the models supported by the Cosmos DB database. We start with `Customer`, `Product`, and `SalesOrder`, and let’s assume we want to add a new model named `Location`. We obviously need to add a class named `Location` with some properties, and then we add `Location` to the model registry. Say `Location` has a property `locationName`. If we say

```csharp
TypeMatchingProperties = new List<string> { “locationName” }
```

this allows us to determine the type of a random JSON document to be `Location` (`ModelRegistry.IdentifyType`). This happens in the generic Cosmos DB change feed handler (`CosmosDbService.cs`, line 159 in the hackathon starter). This way, we use a single, generic change feed handler which can manage any model registered in the model registry. Note that the implicit requirement in all this is that we can’t have identical property sets registered as `TypeMatchingProperties` in the model registry.
 
`NamingProperties` is used to dynamically determine the name of an entity, and is used in `CosmosDbService` line 175, to build that name via reflection. At this point, this name is only used for logging purposes (see lines 172 and 176 from `AzureCognitiveSearchVectorMemory.cs`).
 
`EmbeddedEntity` is used to mark a model that is embedabble via Cog Search vector indexes. As you can see, all our models are embedded entities (`Customer`, `Product`, `SalesOrder`). Any new entity added to the project should also derive from `EmbeddedEntity`. This ensures that all our models have the vector property (for obvious reasons) and a canonical entity name stored in `entityType__` (the name mangling is there to avoid any potential collision with models that do have an `entityType` property).
 
`entityType__` is marked as filterable and facetable, to allow the basic building blocks of Cog Search faceted queries (e.g., filter for `Product` only).
 
`entityType__` is also marked as `EmbeddingField`. This attribute is used by `EmbeddingUtility` when determinig the text version of any entity, right before vectorizing it (`AzureCognitiveSearchVectorMemory`, line 153). `EmbeddingField` defines the prefix used when serializing the property value to text (e.g., the `entityType__` value will always be serialized as “Entity (object) type: <type>” – this allows a more RAG-friendly approach to determining the text representation of entities that are vectorized).
 
`EmbeddingUtility` is used in two important steps:
 
1.	When embedding items intercepted by the Cosmos Db change feeds (in this case we call `Transform(object item, ….)` – we already know the type of the object).
2.	When adding items to the prompt. These items are retrieved by the `TextEmbeddingObjectMemorySkill` in `SemanticKernelRAGService` (line 138) as strings. We need to re-hydrate these to objects and then represent them using the `EmbeddingField`-based approach outlined above. In this case we call `EmebddingUtility.Transform(string item, Dictionary<string, Type> types …)`. This is another important use of the Model Registry.
 
 
Except for `entityType__`, the only other property marked as filterable and facetable is `categoryName` in Product. We did this on purpose, to allow a potential future exercise of playing with the atrributed-driven Cog Search index metadata definitios.

We use two types of RAG memories: long-term and short-term. 
 
Long-term memories are stored in persisted indexes (like the Cog Search vector index) and are considered to be relatively slowly changing (e.g., Cosmos Db items are stored there, since they will not change often, and thus, the vector index itself does not change often). Long-term memories are “recalculated” only if and when needed.
 
Short-term memories are short-lived and stored in volatile memory stores (like the `ShortTermVolatileMemoryStore`). They are refreshed every time the service restarts (or even more often, based on a schedule, for example – this is not implemented, left as an exercise for the reader). The reason this happens is because the information they contain is short-lived itself (e.g., analytical results of Cog Search faceted queries, which might need to be refreshed on a regular basis, to keep them up to date with the content of the index).
 
Our main skill that works with memories is `TextEmbeddingObjectMemorySkill` and it makes use of both the long-term memory store (Cog Search vector index) and the short-term memory store (in-memory, volatile memories).
 
The `RecallAsync` method of `TextEmbeddingObjectMemorySkill` uses a simplistic approach to combine long-term and short-term memories. It takes up to limit items of each, provided the vector distance is above relevance (which by default is 0.7). It then puts the long-term ones first, followed by short-term ones.
 
We use this to showcase the two important flavors of memories (long-term and short-term). Clearly, way better approaches can be used to combine the two, but that is an exercise left to the reader.

