# Solution for Challenge 03

As the team evaluates the solution features, they should walk through the functionality of the web application. They will need to run the React application locally several times as they work through the code completion challenges. They will also need to be able to run the entire solution locally to test their changes before deploying them to Azure.

## Code completion challenges

### CosmosDbChangeFeedService.cs

```csharp
private async Task ProcessCustomerViewChangeFeedHandler(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<JObject> input,
    CancellationToken cancellationToken)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    using var logScope = _logger.BeginScope("Cosmos DB Change Feed Processor: ProcessCustomerViewChangeFeedHandler");

    _logger.LogInformation("Cosmos DB Change Feed Processor: Processing {count} changes...", input.Count);

    await Parallel.ForEachAsync(input, cancellationToken, async (record, token) =>
    {
        try
        {
            // This method of writing the document from the change feed directly to the customerTransactions container
            // with no modifications to the document is called a "passthrough" pattern. We are taking this step to
            // implement a CQRS pattern, where the customerTransactions container is the "read" side of the pattern and
            // the transactions container is the "write" side of the pattern. This pattern is used to optimize the
            // read and write operations for each container, minimizing impact on potentially heavy write operations.

            // TODO: Uncomment the following line and complete the code to upsert (insert/update) the document to the
            // customerTransactions container via the customer repository.
            await _customerRepository.UpsertItem(record);
        }
        catch (Exception ex)
        {
            //Should handle DLQ
            _logger.LogError(ex.Message, ex);
        }
    });
}
```

### MemberRepository.cs

```csharp
public async Task<int> PatchMember(Member member, string memberId)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    JObject obj = JObject.FromObject(member);

    var ops = new List<PatchOperation>();

    foreach (JToken item in obj.Values())
    {
        // TODO: Uncomment the following line and complete the code to skip the following paths:
        //       "id", "memberId", "type", and empty strings.
        //       Hint: Evaluate the item.Path and item.ToString() values.
        if (item.Path is "id" or "memberId" or "type" || string.IsNullOrEmpty(item.ToString()))
            continue;

        // TODO: Uncomment the following line and complete the code to add the patch operation to the list.
        //       Hint: Create a new PatchOperation with the path and value to add the item.
        ops.Add(PatchOperation.Add($"/{item.Path}", item.ToString()));
    }

    if (ops.Count == 0)
        return 0;

    var response = await Container.PatchItemAsync<Member>(memberId, new PartitionKey(memberId), ops);

    return ops.Count;
}
```

### TransactionRepository.cs

```csharp
public async Task<(AccountSummary? accountSummary, HttpStatusCode statusCode, string message)> ProcessTransactionTBatch(Transaction transaction)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    var pk = new PartitionKey(transaction.accountId);

    var responseRead = await ReadItem<AccountSummary>(transaction.accountId, transaction.accountId);
    var account = responseRead.Resource;

    if (account == null)
    {
        return new(null, HttpStatusCode.NotFound, "Account not found!");
    }

    // TODO: Create a business rule to check if the transaction type is "debit" and throw an
    //       exception if the transaction amount is greater than the account balance plus the overdraft limit.
    if (transaction.type.ToLowerInvariant() == Constants.DocumentTypes.TransactionDebit)
    {
        // TODO: Uncomment the following lines and complete the code to check if the transaction amount is
        //       greater than the account balance plus the overdraft limit.
        if ((account.balance + account.overdraftLimit) < transaction.amount)
        {
            return new(null, HttpStatusCode.BadRequest, "Insufficient balance/limit!");
        }
        else
        {
            account.balance -= transaction.amount;
        }
    }

    var batch = Container.CreateTransactionalBatch(pk);

    batch.PatchItem(account.id,
        new List<PatchOperation>()
        {
            PatchOperation.Increment("/balance", transaction.type.ToLowerInvariant() == Constants.DocumentTypes.TransactionDebit ? -transaction.amount : transaction.amount)
        },
        new TransactionalBatchPatchItemRequestOptions()
        {
            IfMatchEtag = responseRead.ETag
        }
    );
    batch.CreateItem<Transaction>(transaction);

    var responseBatch = await batch.ExecuteAsync();

    if (responseBatch.IsSuccessStatusCode)
    {
        account = responseBatch.GetOperationResultAtIndex<AccountSummary>(0).Resource;
        return new(account, HttpStatusCode.OK, string.Empty);
    }
    else if (responseBatch.StatusCode == HttpStatusCode.PreconditionFailed)
        return new (null, HttpStatusCode.PreconditionFailed, string.Empty);
    else
        return new (null, HttpStatusCode.BadRequest, string.Empty);
}
```

## Running the solution locally

The following steps will help the team run the React application locally:

1. Navigate to the `ui` folder in a terminal/command prompt window.
2. Run `npm install` to restore the packages.
3. Run `npm run dev`.
4. Open localhost:3000 in a web browser.

For the backend components, they will need to do the following to run locally:

> [!NOTE]
> The machine on which the deployment scripts executed should already have the appropriate application settings configured for each project (`appsettings.Development.json`). If the team is using a different machine, they will need to configure the application settings manually. Team members should be able to get the appropriate values from the person who deployed the solution.

1. Start Docker Desktop.
2. Navigate to the `src` folder and open `CorePayments.sln` in Visual Studio.
3. Ensure the `appsettings.Development.json` files have been created for each project. If not, create them and copy the contents from the `appsettings.json` files (or `appsettings.Development.template.json` if exists) and update the values as appropriate.
4. Ensure the user has RBAC access to Cosmos DB as described in the [solution notes](../../solution-notes.md).
5. Ensure the user is signed in to Azure from Visual Studio.
6. Right-click the solution and select "Set Startup Projects..."
   1. Select "Multiple startup projects".
   2. Set the action for the following projects to "Start":
      - `CorePayments.WorkerService`
      - `CorePayments.WebAPI`
   3. Click "OK".
7. Press F5 to start debugging.

## Exploring the solution

> The team will be able to complete all of the functions below _after_ they have finished the code completion challenges.

Run the solution locally and walk through the following:

The application frontend is a React JavaScript single-page app (SPA) that simulates a business-facing web application for managing members, accounts, and transactions.

1. Browse to http://localhost:3000.
2. Members page:
    1. Observe how paging works (`CosmosDbRepository.PagedQuery` method), which uses continuation tokens to efficiently fetch additional records from Cosmos DB. This is more performant than traditional paging methods, especially once you've paged through a large number of records.
    2. Click `Details` for a member and select `Edit` to display the form. We use a Patch operation instead of an update in the back end as opposed to an update on the Cosmos DB record. This is a more efficient operation. Plus, it is easier to reconcile conflicts if more than one person applies updates on a record simultaneously.
    3. Select `View Accounts`.
    4. Click on `Assign Account` and search for an existing account number. Explore how global index works (`GlobalIndexRepository.cs`) and how there is a many-to-many relationship between members and accounts. Think of family members in a household who may have access to a shared account and have their own accounts that only they can access. Or a business account managed by multiple users. Use `Remove Account Assignment`.
    5. Click on `View Details` to display a modal containing account details. This is a good place to mention the Overdraft Limit and how the overdraft rule applies to new transactions.
    6. Click on `View Transactions`.
    7. Click on `New Transaction` to display the modal and enter a new transaction. (To force a refresh, you can select `View Transactions` for another user, then `View Transactions` on the same user once more)
    8. Try entering a transaction that exceeds the account balance + overdraft limit. It should fail to post.

> We'll explore the `Analyze Transactions` option in the next challenge after completing the Semantic Kernel integration.
