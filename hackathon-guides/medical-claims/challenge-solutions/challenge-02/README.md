# Solution for Challenge 02

In this challenge, the team should look at the deployed assets and discover Azure Synapse Analytics within the Azure resource group. If the team is sharing a single resource group, the owner will need to assign permissions to various resources to the team members.

The following script grants access to the workspace via Azure Cloud Shell or Azure CLI, where `AZURE_AD_PRINCIPAL_ID` is the Azure AD principal ID of the user to be granted access and `SYNAPSE_WORKSPACE_NAME` is the name of the Synapse workspace:

    ```bash
    az synapse role assignment create --workspace-name SYNAPSE_WORKSPACE_NAME --role 6e4bf58a-b8e1-4cc3-bbf9-d73143322b78 --assignee AZURE_AD_PRINCIPAL_ID
    ```

> Pre-generated data can be found in a publicly accessible Azure Blob Storage account (https://solliancepublicdata.blob.core.windows.net/medical-claims).

As part of the deployment, the team should have created a Synapse workspace. The Synapse workspace contains a number of assets, some linked services, datasets, and a pipeline. The team should explore the assets and understand how they are used in the solution.

Ultimately, they should execute the pipeline to load the data into the Azure Cosmos DB containers. The execution time should not exceed 5 minutes.

> [!NOTE]
> Allow the team to figure this out on their own. The instructins they see within Challenge 02 doe not explicitly tell them to load data using Synapse Analytics, though there is a link to learn more about Synapse Analytics pipelines under the Resources section. If they are stuck, you can point them to the Synapse Analytics workspace and the pipeline.
