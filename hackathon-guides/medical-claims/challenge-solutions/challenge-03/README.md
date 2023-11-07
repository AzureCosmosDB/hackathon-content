# Solution for Challenge 03

As the team evaluates the solution features, they should walk through the functionality of the web application. They will need to run the React application locally several times as they work through the code completion challenges. They will also need to be able to run the entire solution locally to test their changes before deploying them to Azure.

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
