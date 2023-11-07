# Solution for Challenge 03

As the team evaluates the solution features, they should walk through the functionality of the web application. They will need to run the React application locally several times as they work through the code completion challenges. They will also need to be able to run the entire solution locally to test their changes before deploying them to Azure.

## Code completion challenges

### CosmosDbChangeFeedService.cs

```csharp
private async Task AssignClaimAdjudicatorChangeFeedHandler(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<ClaimHeader> input,
    CancellationToken cancellationToken)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    using var logScope = _logger.BeginScope("CosmosDbTrigger: AssignClaimAdjudicator");

    // Get all claims that has initial status.
    var items = input.Where(i => i.Type == ClaimHeader.EntityName &&
                                 i.ClaimStatus is ClaimStatus.Initial or ClaimStatus.Resubmitted or ClaimStatus.Proposed);

    foreach (var doc in items)
    {
        _logger.LogInformation("Processing document " + doc.Id);

        // Get claim detail to update.
        var claim = await _claimRepository.GetClaim(doc.ClaimId, doc.AdjustmentId);
        if (claim == null) throw new ArgumentException($"Claim '{doc.ClaimId}' missing", nameof(doc.ClaimId));

        claim.ModifiedBy = "ChangeFeed/Adjudication";

        switch (doc.ClaimStatus)
        {
            case ClaimStatus.Initial:
            case ClaimStatus.Resubmitted:
            {
                // Assign claim to an adjudicator based on business rule.
                claim = await _coreBusinessRule.AssignClaim(claim);
                break;
            }

            case ClaimStatus.Proposed:
            {
                // Adjudicate claim.
                //Create a copy of the claim and name it as "Proposed" claim.
                var proposedClaim = new ClaimHeader(claim);
                // TODO: Following the same pattern as above for Resubmitted claims, uncomment and complete the following line to adjudicate the claim.
                (claim, var adjudicatorChanged) = await _coreBusinessRule.AdjudicateClaim(claim);
                // TODO: Uncomment and complete the following lines to raise an Event Hub event for the adjudicator change, passing in the proposedClaim as the event payload:
                if (adjudicatorChanged)
                {
                    // Raise event to notify adjudicator change.
                    await _eventHub.TriggerEventAsync(proposedClaim, Constants.EventHubTopics.AdjudicatorChanged);
                }
                break;
            }
        }

        // Update claim header and details in Claim container.
        await _claimRepository.UpdateClaim(claim);
    }
}
```

```csharp
private async Task ClaimCompleteChangeFeedHandler(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<ClaimDetail> input,
    CancellationToken cancellationToken)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    using var logScope = _logger.BeginScope("CosmosDbTrigger: ClaimComplete");

    try
    {
        foreach (var claim in input.Where(c =>
                     c.Type == ClaimDetail.EntityName &&
                     c.ClaimStatus is ClaimStatus.Approved or ClaimStatus.Denied))
        {
            switch (claim.ClaimStatus)
            {
                case ClaimStatus.Approved:
                    //TODO: Increment the member totals by calling a method on the Member repository.
                    //      Pass to the method the MemberId, a count of 1, and the total amount from the claim.
                    await _memberRepository.IncrementMemberTotals(claim.MemberId, 1, claim.TotalAmount);
                    await _eventHub.TriggerEventAsync(claim, Constants.EventHubTopics.Approved);
                    break;
                case ClaimStatus.Denied:
                    await _eventHub.TriggerEventAsync(claim, Constants.EventHubTopics.Denied);
                    break;
            }

            _logger.LogInformation($"Claim {claim.ClaimId} published to EventHub/{claim.ClaimStatus}");
        }
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, $"Failed to publish ClaimComplete events");
        throw;
    }
}
```

```csharp
private async Task ClaimUpdatedChangeFeedHandler(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<ClaimHeader> input,
    CancellationToken cancellationToken)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    using var logScope = _logger.BeginScope("CosmosDbTrigger: ClaimUpdated");

    var headers = input.Where(i => i.Type == ClaimHeader.EntityName);

    foreach (var claim in headers)
    {
        if (!string.IsNullOrEmpty(claim.MemberId))
        {
            // TODO: Upsert the claim in the Member repository.
            await _memberRepository.UpsertClaim(claim);
            _logger.LogInformation($"Updating ClaimHeader/{claim.ClaimId}/{claim.AdjustmentId} for Member/{claim.MemberId}");
        }

        if (!string.IsNullOrEmpty(claim.AdjudicatorId))
        {
            // TODO: Upsert the claim in the Adjudicator repository.
            await _adjudicatorRepository.UpsertClaim(claim);
            _logger.LogInformation($"Updating ClaimHeader/{claim.ClaimId}/{claim.AdjustmentId} for Adjudicator/{claim.AdjudicatorId}");
        }
    }
}
```

### CoreBusinessRule.cs

```csharp
public async Task<ClaimDetail> AssignClaim(ClaimDetail claim)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */

    // If claim has no member, set status to Pending.
    if (string.IsNullOrEmpty(claim.MemberId))
    {
        claim.ClaimStatus = ClaimStatus.Pending;
        return claim;
    }

    // Check if member has active coverage.
    // If member has active coverage, check if claim is still covered based on dates.
    // If claim is not valid, set status to Rejected.
    var coverages = await _memberRepository.GetMemberCoverage(claim.MemberId);
    if (!coverages.Any(c => c.StartDate <= claim.FilingDate && c.EndDate >= claim.FilingDate))
    {
        claim.ClaimStatus = ClaimStatus.Denied;
        claim.Comment = "[Automatic] Rejected: Coverage expired or missing";
        return claim;
    }

    // If claim's total amount is less than 200, set status to Approved.
    // TODO: If the claim's TotalAmount value is less than the AutoApproveThreshold in the BusinessRulesOptions,
    //       set the claim status to Approved and return the claim.
    if (claim.TotalAmount < _options.Value.AutoApproveThreshold)
    {
        claim.ClaimStatus = ClaimStatus.Approved;
        claim.Comment = $"[Automatic] Approved: Less than threshold of {_options.Value.AutoApproveThreshold}";
        return claim;
    }

    // Select random adjudicator.
    if (string.IsNullOrEmpty(claim.AdjudicatorId))
    {
        Adjudicator adjudicator;
        if (_options.Value.DemoMode)
        {
            var demoAdjudicatorId = _options.Value.DemoAdjudicatorId;
            var demoManagerAdjudicatorId = _options.Value.DemoManagerAdjudicatorId;

            if (!string.IsNullOrWhiteSpace(demoAdjudicatorId) && !string.IsNullOrWhiteSpace(demoManagerAdjudicatorId))
            {
                var randomIndex = random.Next(0, 2);

                var selectedAdjudicatorId = randomIndex == 0 ? demoAdjudicatorId : demoManagerAdjudicatorId;

                adjudicator = await _adjudicatorRepository.GetAdjudicator(selectedAdjudicatorId) ?? await _adjudicatorRepository.GetRandomAdjudicator();
            }
            else if (!string.IsNullOrWhiteSpace(demoAdjudicatorId))
            {
                adjudicator = await _adjudicatorRepository.GetAdjudicator(demoAdjudicatorId) ?? await _adjudicatorRepository.GetRandomAdjudicator();
            }
            else if (!string.IsNullOrWhiteSpace(demoManagerAdjudicatorId))
            {
                adjudicator = await _adjudicatorRepository.GetAdjudicator(demoManagerAdjudicatorId) ?? await _adjudicatorRepository.GetRandomAdjudicator();
            }
            else
            {
                adjudicator = await _adjudicatorRepository.GetRandomAdjudicator();
            }
        }
        else
        {
            adjudicator = await _adjudicatorRepository.GetRandomAdjudicator();
        }
        
        if (adjudicator != null)
        {
            // Set Adjudicator to the claim and set status to Assigned.
            claim.AdjudicatorId = adjudicator.AdjudicatorId;
            claim.ClaimStatus = ClaimStatus.Assigned;
            claim.Comment = $"[Automatic] Assigned: Automatically assigned to Adjudicator {adjudicator.AdjudicatorId} ({adjudicator.Role})";
        }
    }

    return claim;
}
```

```csharp
public async Task<(ClaimDetail, bool)> AdjudicateClaim(ClaimDetail claim)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */

    // TODO: Retrieve the Adjudicator from the AdjudicatorRepository based on the AdjudicatorId assigned to the claim:
    var adjudicator = await _adjudicatorRepository.GetAdjudicator(claim.AdjudicatorId);
    var adjudicatorChanged = false;

    // TODO: Check if the adjudicator has a Manager role. If so, set status to Approved, add a comment to the claim, and return the claim and adjudicatorChanged values.
    if (adjudicator?.Role == AdjudicatorRole.Manager)
    {
        claim.ClaimStatus = ClaimStatus.Approved;
        claim.Comment = "[Automatic] Approved: Manager Proposed adjustment";
        return (claim, adjudicatorChanged);
    }

    if (claim.LastAmount.HasValue)
    {
        // TODO: Check if the difference between the LastAmount and TotalAmount is less than or equal to the RequireManagerApproval threshold.
        // If so, set status to Approved, add a comment to the claim, and return the claim and adjudicatorChanged values.
        decimal difference = Math.Abs(claim.LastAmount.Value - claim.TotalAmount);
        if (difference <= _options.Value.RequireManagerApproval)
        {
            claim.ClaimStatus = ClaimStatus.Approved;
            claim.Comment = $"[Automatic] Approved: Proposed adjustment below approval threshold {_options.Value.RequireManagerApproval}";
            return (claim, adjudicatorChanged);
        }
    }

    Adjudicator manager;
    if (_options.Value.DemoMode && !string.IsNullOrWhiteSpace(_options.Value.DemoManagerAdjudicatorId))
    {
        // TODO: If DemoMode is enabled and a DemoManagerAdjudicatorId is set, retrieve the Adjudicator from the AdjudicatorRepository based on the DemoManagerAdjudicatorId:
        manager = await _adjudicatorRepository.GetAdjudicator(_options.Value.DemoManagerAdjudicatorId) ?? await _adjudicatorRepository.GetRandomAdjudicator("Manager");
    }
    else
    {
        // TODO: Retrieve a random Adjudicator from the AdjudicatorRepository with a role of "Manager":
        manager = await _adjudicatorRepository.GetRandomAdjudicator("Manager");
    }
    
    // TODO: If a manager was found, set the claim's PreviousAdjudicatorId to the claim's AdjudicatorId, set the claim's AdjudicatorId to the manager's AdjudicatorId,
    if (manager == null)
        throw new NullReferenceException("Unable to find an appropriate manager to assign approval to");

    claim.PreviousAdjudicatorId = claim.AdjudicatorId;
    claim.AdjudicatorId = manager.Id;
    claim.ClaimStatus = ClaimStatus.ApprovalRequired;
    claim.Comment = "[Automatic] Reassigned: Adjustment requires manager approval";

    adjudicatorChanged = claim.PreviousAdjudicatorId != claim.AdjudicatorId;

    return (claim, adjudicatorChanged);
}
```

### MemberRepository.cs

```csharp
public async Task<Member> IncrementMemberTotals(string memberId, int count, decimal amount)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    var response = await Container.PatchItemAsync<Member>(memberId, new PartitionKey(memberId),
        patchOperations: new[]
        {
            // TODO: Increment the approvedCount and approvedTotal properties with patch operations.
            //       Convert the amount to a double using the decimal.ToDouble method.
            PatchOperation.Increment("/approvedCount", count),
            PatchOperation.Increment("/approvedTotal", decimal.ToDouble(amount)),
            PatchOperation.Replace("/modifiedBy", "System/IncrementTotals"),
            PatchOperation.Replace("/modifiedOn", DateTime.UtcNow), 
        });

    return response.Resource;
}
```

### ClaimRepository.cs

```csharp
public async Task<ClaimHeader> CreateClaim(ClaimDetail detail)
{
    /* TODO: Challenge 3.
    * Uncomment and complete the following lines as instructed.
    */
    detail.CreatedOn = detail.ModifiedOn = DateTime.UtcNow;

    var header = new ClaimHeader(detail);

    // TODO: Create a transactional batch to create the header and detail items.
    var batch = Container.CreateTransactionalBatch(new PartitionKey(detail.ClaimId));
    batch.CreateItem(header);
    batch.CreateItem(detail);

    // TODO: Uncomment the following lines.
    using (var response = await batch.ExecuteAsync())
    {
        if (!response.IsSuccessStatusCode)
        {
            throw new Exception(response.ErrorMessage); // TODO: Better than this
        }

        return response.GetOperationResultAtIndex<ClaimHeader>(0).Resource;
    };
}
```

## Running the solution locally

The following steps will help the team run the React application locally:

1. Navigate to the `ui/medical-claims-ui` folder in a terminal/command prompt window.
2. Run `npm install` to restore the packages.
3. Run `npm run dev`.
4. Open localhost:3000 in a web browser.

For the backend components, they will need to do the following to run locally:

> [!NOTE]
> The machine on which the deployment scripts executed should already have the appropriate application settings configured for each project (`appsettings.Development.json`). If the team is using a different machine, they will need to configure the application settings manually. Team members should be able to get the appropriate values from the person who deployed the solution.

1. Start Docker Desktop.
2. Navigate to the `src` folder and open `CoreClaims.sln` in Visual Studio.
3. Ensure the `appsettings.Development.json` files have been created for each project. If not, create them and copy the contents from the `appsettings.json` files (or `appsettings.Development.template.json` if exists) and update the values as appropriate.
4. Ensure the user has RBAC access to the Azure resources (Cosmos DB, Event Hubs, etc.) as described in the [solution notes](../../solution-notes.md).
5. Ensure the user is signed in to Azure from Visual Studio.
6. Right-click the solution and select "Set Startup Projects..."
   1. Select "Multiple startup projects".
   2. Set the action for the following projects to "Start":
      - `CoreClaims.WorkerService`
      - `CoreClaims.WebAPI`
   3. Click "OK".
7. Press F5 to start debugging.

## Exploring the solution

> The team will be able to complete all of the functions below _after_ they have finished the code completion challenges.

Run the solution locally and walk through the following:

The application frontend is a React JavaScript single-page app (SPA) that simulates a business-facing web application for managing members, payers, providers, and claims.

1. Browse to http://localhost:3000.
2. Start on the home page, which shows the Business Rules fetched from the API
   1. The Auto-Approve Threshold is set to $200. This means that if the totalAmount value is less than $200 (configurable) it should be automatically Approved. If the totalAmount value is greater than $200, it should be Assigned.
   2. If the adjudicator proposes a claim and applies discounts on the line items that exceed $500, it must be approved by a manager. The rules engine reassigns the claim to a manager in this case. However, if a manager proposes discounts that exceed the threshold, it is automatically approved.
   3. If the member this claim belongs to doesn't have a Coverage record that is active for the filingDate, the claim should be Rejected.
3. Expand the left menu bar to show how to display the labels for the page links. Collapse the menu when done.
4. Show the Members page.
    1. Click on the Approved Count header on the grid twice to sort in reverse order. This lets us find a member with multiple approved claims.
    2. Click on Details of the first member.
    3. Click on View Coverage of the first member and make note of the coverage dates. Any claims that come through the system after the End Date will automatically be denied.
    4. Click on View Claims for the first member. Note the claim status of each. Click Details next to a Denied claim. Make note of the comment that it was automatically rejected due to expired or missing coverage.
    5. Click on View History next to an Approved claim. Take note of the Initial claim status on the bottom and that it was modified by Synapse/Ingestion from our initial data load. Then look at the latest entry as to why it was approved.
5. Navigate to the Providers page and show the list. These are medical providers, like hospitals, doctors, and clinics.
6. Navigate to the Payers page and show the list. These are insurance companies that pay the claims.
7. Navigate to the Adjudicators page.
    1. Select the Non-Manager and Manager tabs. These tabs are for demo purposes only so we do not have to sign in with two different accounts to interact with the site as either an adjudicator or a adjudicator manager.
    2. Select Details for a claim on the Non-Manager tab that has a Total Amount of over $500.
        1. Select Acknowledge Claim Assignment and make note of the Claim Id.
        2. Navigate to the last page of the claims list. The last item will be the claim that we just acknowledged. Notice that there was a slight delay. That’s because all actions are processed asynchronously, with claims data traversing through the API, change feed, and in some cases, an Event Hub and their subsequent processors. Select Details on the Acknowledged claim.
        3. **Note:** The Make Recommendation button will not yet function. The team will complete the steps to wire this up in the next challenge.
        4. Apply discounts on the line items whose total exceeds $500, then select Propose Claim and enter comments, then click Save. Notice that the Claim Status is now Proposed.
            1. Some technical background on this process: When adding new claims, they are currently assigned to an Adjudicator. When this happens, a `ClaimHeader` document is stored in the `Adjudicator` container. That container has the `adjudicatorId` as its partition key. As we know, it is not possible to change the partition key value of a document once it's been saved to the container. Because of this, if the claim gets reassigned to a manager, according to the core business rules, the claim details record is updated, triggering a downstream update of the `ClaimHeader` document stored in the `Adjudicator` container, this time **with a new `adjudicatorId`** value (the value of the assigned manager). In some cases, this value might not change, like if the claim was originally assigned to a manager. But in most cases, it does. When the adjudicator is reassigned, we get a **new** `ClaimHeader` record stored in the `Adjudicator` container from the `AdjudicatorRepository.UpsertClaim` method that gets called by the `ClaimUpdated` Change Feed trigger. Ideally, we would create a new batch transaction to delete the previous adjudicator's document and upsert the new adjudicator's document. This cannot be done since they live in different logical partitions. To get around this, we implement a Transactional Outbox pattern. This is why we publish the previous adjudicator's `ClaimHeader` document to the `AdjudicatorChanged` event topic before upserting the new adjudicator's `ClaimHeader` document. The downstream EventHub processor executes an idempotent method to delete the previous adjudicator's `ClaimHeader` document from the `Adjudicator` container if it exists.
         5. After a few seconds, click on the Manager tab to switch to the manager’s view. Click on the Last Adjudicated Date twice to sort in descending order. You should see the proposed claim at the top with an ApprovalRequired claim status. Make note of the Last Amount and Total Amount values for this claim in the grid, then select Details.
         6. View the Comment and the fact that it was automatically reassigned.
         7. Select Approve Claim, add comments, then click Save.
         8. Click a few times on the Last Adjudicated Date header in the grid until you see the claim at the top with an Approved status. Select Details.
         9. Look at the comments automatically applied.
         10. Select View History on the claim and walk through the entire lifecycle of the claim, starting at the very bottom when it was initialized. You will also see where manual comments were added by the adjudicator and manager.
