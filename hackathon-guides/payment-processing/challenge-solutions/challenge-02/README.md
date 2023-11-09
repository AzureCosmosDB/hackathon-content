# Solution for Challenge 02

This challenge requires the team to complete the data loading code that generates customer and transaction data. This gives them a chance to get familiar with the Cosmos DB SDK and understand the data model, including the global index.

## Code completion challenges

### account-generator.Program.cs

```csharp
public static async Task MainAsync(string[] args)
{
    /* TODO: Challenge 2.
    * Uncomment and complete the following lines as instructed.
    */
    var configuration = new ConfigurationBuilder().SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("local.settings.json")
        .AddJsonFile("settings.json", optional: true, reloadOnChange: true)
        .AddCommandLine(args, new Dictionary<string, string>
        {
            {"-m", $"{nameof(GeneratorOptions)}:{nameof(GeneratorOptions.RunMode)}"},
            {"-s", $"{nameof(GeneratorOptions)}:{nameof(GeneratorOptions.SleepTime)}"},
            {"-c", $"{nameof(GeneratorOptions)}:{nameof(GeneratorOptions.BatchSize)}"},
            {"-v", $"{nameof(GeneratorOptions)}:{nameof(GeneratorOptions.Verbose)}"}
        })
        .AddEnvironmentVariables()
        .Build();

    GeneratorOptions options = new();
    configuration.GetSection(nameof(GeneratorOptions))
        .Bind(options);

    Console.WriteLine("To STOP press CTRL+C...");

    Console.CancelKeyPress += Console_CancelKeyPressHandler;

    cosmosClient = new CosmosClient(configuration["CosmosDbConnectionString"],
        new CosmosClientOptions() {AllowBulkExecution = true, EnableContentResponseOnWrite = false});

    // TODO: Instantiate the transactionsContainer and membersContainer with new Container objects from the CosmosClient.
    //       Use the "payments" database and "transactions", "members", and "globalIndex" containers.
    transactionsContainer = cosmosClient.GetContainer("payments", "transactions");
    membersContainer = cosmosClient.GetContainer("payments", "members");
    globalIndexContainer = cosmosClient.GetContainer("payments", "globalIndex");

    // Generate Members if they don't already exist:
    var memberList = await CreateMembersAsync();

    var tasks = new List<Task>();

    try
    {
        for (var i = 1; i <= 5; i++)
        {
            tasks.Add(LoadAsync(i, options, memberList));
        }

        Task.WhenAll(tasks).GetAwaiter().GetResult();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }

    Console.WriteLine("Completed generating data.");
    cosmosClient.Dispose();
}
```

```csharp
private static async Task<List<Member>> CreateMembersAsync()
{
    /* TODO: Challenge 2.
    * Uncomment and complete the following lines as instructed.
    */    
    var memberList = new List<Member>();
    var query = "SELECT * FROM c";
    var queryDefinition = new QueryDefinition(query);
    var resultIterator = membersContainer.GetItemQueryIterator<Member>(queryDefinition);

    while (resultIterator.HasMoreResults)
    {
        var response = await resultIterator.ReadNextAsync();
        memberList.AddRange(response);
    }

    if (memberList.Count > 0)
    {
        Console.WriteLine("Skipping Member record generation since records already exist.");
        return memberList;
    }

    for (var i = 0; i <= 350; i++)
    {
        var memberId = Guid.NewGuid().ToString();
        var memberFaker = new Faker<Member>()
            .RuleFor(u => u.memberId, (f, u) => memberId)
            .RuleFor(u => u.firstName, (f, u) => f.Name.FirstName())
            .RuleFor(u => u.lastName, (f, u) => f.Name.LastName())
            .RuleFor(u => u.email, (f, u) => f.Internet.Email())
            .RuleFor(u => u.phone, (f, u) => f.Phone.PhoneNumber())
            .RuleFor(u => u.address, (f, u) => f.Address.StreetAddress())
            .RuleFor(u => u.city, (f, u) => f.Address.City())
            .RuleFor(u => u.state, (f, u) => f.Address.State())
            .RuleFor(u => u.zipcode, (f, u) => f.Address.ZipCode("#####"))
            .RuleFor(u => u.country, (f, u) => "USA")
            .RuleFor(u => u.type, (f, u) => Constants.DocumentTypes.Member)
            .RuleFor(u => u.memberSince, (f, u) => f.Date.Past(20));

        await _pollyRetryPolicy.ExecuteAsync(async () =>
        {
            // TODO: Finish the code below to Upsert the Member item in the Members container.
            //       Set the partition key to the memberId. It is important that you upsert, not insert.
            var member = memberFaker.Generate();
            await membersContainer.UpsertItemAsync(member, new PartitionKey(memberId));
            memberList.Add(member);
            // Create a global index lookup for this member.
            var globalIndex = new GlobalIndex
            {
                partitionKey = memberId,
                targetDocType = Constants.DocumentTypes.Member,
                id = memberId
            };
            await globalIndexContainer.UpsertItemAsync(globalIndex, new PartitionKey(globalIndex.partitionKey));
        });
    }

    Console.WriteLine("Finished generating Members.");
    return memberList;
}
```

## Running the account generator

> [!NOTE]
> The machine on which the deployment scripts executed should already have the appropriate application settings configured for the account generator (`local.settings.json`). If the team is using a different machine, they will need to configure the application settings manually. Team members should be able to get the appropriate values from the person who deployed the solution.

1. Navigate to the `src` folder and open `CorePayments.sln` in Visual Studio.
2. Ensure the `local.settings.json` file has been created for the account generator project. If not, copy the `local.settings.template.json` file and add the `CosmosDbConnectionString` value.
3. Right-click the `account-generator` project, select `Debug`, then `Start new instance`.
4. The account generator should start running and generating data. It will take a few minutes to generate all the data. The team can monitor the progress in the console window.
