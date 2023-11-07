# Solution for Challenge 04

This challenge has the team configure Semantic Kernel to provide recommendations to a claims adjudicator on whether to approve or reject a claim, based on the claims data and business rules.

After completing the code challenges, the team should be able to run the solution locally and walk through the following:

1. Navigate to the Adjudicators page.
    1. Select Details for a claim on the Non-Manager tab that has a Total Amount of over $500.
    2. Click on the Make Recommendation button, then click on “Ask for Recommendation”. This triggers a call into the Azure OpenAI completion model, along with the claim information and customized system prompt, orchestrated by Semantic Kernel. Read the response and close the dialog.

Does the recommendation make sense? If not, the team should review the system prompt and make adjustments as necessary. The provided system prompt works _reasonably_ well in most cases, but it may need to be tweaked to improve the results. This is where the team can get creative and experiment with different prompts to see what works best. Where they can make the most impact to the system prompt is in the `Review Rules` section. The team can add additional rules to the system prompt to improve the results. For example, if the claim is for a member that is not covered, the team can add a rule to reject the claim. If the claim is for a member that is covered, the team can add a rule to approve the claim. The team can also add rules to approve or reject claims based on the total amount of the claim.

> [!NOTE]
> At this point, the team will likely discover that Semantic Kernel does not handle mathematical operations very well. Improving this capability is a stretch goal and something they can tackle in the next challenge.
