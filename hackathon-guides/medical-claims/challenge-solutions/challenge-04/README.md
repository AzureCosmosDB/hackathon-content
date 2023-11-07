# Solution for Challenge 04

This challenge has the team configure Semantic Kernel to provide recommendations to a claims adjudicator on whether to approve or reject a claim, based on the claims data and business rules.

## Code completion challenges

### RulesEngine.cs

The team should add the following code to the `RulesEngine` class:

```csharp
public async Task<string> ReviewClaim(ClaimDetail claim)
{
    /* TODO: Challenge 4.
    * Uncomment and complete the following lines as instructed.
    */
    await Task.CompletedTask;

    // TODO: Uncomment the code below and instantiate a new KernelBuilder.
    var builder = new KernelBuilder();

    builder.WithAzureChatCompletionService(
             _settings.OpenAICompletionsDeployment,
             _settings.OpenAIEndpoint,
             _settings.OpenAIKey);

    // TODO: Create a new kernel instance from the builder.
    var kernel = builder.Build();

    string skPrompt = @"
    You are an insurance claim review agent. You are provided data about a claim in JSON format and rules to evaluate the claim and classify it with a review result. 
    For a given claim you must specify one of the following results:
    - [No Action]
    - [Approve]
    - [Deny]
    - [Send to supervisor]

    First summarize the claim by providing the ""TotalAmount"" and the sum of ""Amount"" values.
    Then evaluate the claim according to the review rules and your summary. You must base your review result only on the rules provided and not use any other source for your decision. 
    You must also explain your reasoning for the review result you provide.

    Evaluate the following rules in order such that all Deny rules are evaluated before Approve rules.

    Review Rules
    - [No Action]: if the value of ""ClaimStatus"" is either 3, 4, 5, or 6 the claim has already been processed so the result is No Action.  
    - [Deny]: Let X = the ""TotalAmount"" value. Let Y = the sum of ""Amount"" values. If X is not equal to Y then the result is Deny.
    - [Approve]: if none of the Deny rules match, approve the claim.
    - [Send to supervisor]: send the claim to the supervisor if you don't know what to do, the applicable rules are contradictory or the rules do not explain what to do in this case.

    For example:

    +++

    [INPUT]
    {
      ""TotalAmount"": 0.91,
      ""LineItems"": [
        {
          ""LineItemNo"": 1,
          ""ProcedureCode"": ""314076"",
          ""Description"": ""lisinopril 10 MG Oral Tablet"",
          ""Amount"": 0.91,
          ""Discount"": 0,
          ""ServiceDate"": ""2023-01-17T21:02:18Z""
        }
      ],
      ""Comment"": ""[Automatic] Approved: Less than threshold of 200"",
      ""PreviousAdjudicatorId"": null,
    }
    [END INPUT]

    Provide your response as completions to the following bullets:
    - Summary:
         The ""TotalAmount"" is 0.91 and the sum of ""Amount"" values is 0.91.
    - Review Result: 
        No Action
    - Reasoning: 
        The value of ""ClaimStatus"" is 3 meaning the claim has already been processed so the result is No Action.


    +++

    [INPUT]
    {
      ""AdjustmentId"": 1,
      ""ClaimStatus"": 1,
      ""TotalAmount"": 231.91,
      ""LineItems"": [
        {
          ""LineItemNo"": 1,
          ""ProcedureCode"": ""314076"",
          ""Description"": ""lisinopril 10 MG Oral Tablet"",
          ""Amount"": 0.91,
          ""Discount"": 0,
          ""ServiceDate"": ""2023-01-17T21:02:18Z""
        }
      ],
      ""Comment"": """",
    }
    [END INPUT]

    Provide your response as completions to the following bullets:
    - Summary:
         The ""TotalAmount"" is 231.91 and the sum of ""Amount"" values is 0.91.
    - Review Result: 
        Deny
    - Reasoning: 
        The ""TotalAmount"" (231.91) is greater than 200.00.

    +++

    [INPUT]
    {{$claim}}
    [END INPUT]

    Provide your response as completions to the following bullets replacing the values in square brackets, and do not include the claim context data in the response:
    - Summary:
        [Your Summary of the claim]
    - Review Result:
        [Your Review Result]
    - Reasoning:
        [Your Review Result Reasoning]
    ";

    // TODO: Complete the code below to create a new semantic function for the review skill.
    // HINT: The first parameter is the semantic kernel prompt created above.
    var reviewer = kernel.CreateSemanticFunction(skPrompt, "review", "ReviewSkill", description: "Review the claim and make approval or denial recommendation.", maxTokens: 2000, temperature: 0.0);

    JsonSerializerOptions ser_options = new()
    {
        WriteIndented = true,
        MaxDepth = 20,
        AllowTrailingCommas = true,
        PropertyNameCaseInsensitive = true,
        ReadCommentHandling = JsonCommentHandling.Skip,
    };

    // TODO: Uncomment the below two lines to create a new context from the kernel and set the claim context to your claim data.
    string claimData = JsonSerializer.Serialize(claim, ser_options);

    var context = kernel.CreateNewContext();
    context["claim"] = claimData;

    // TODO: Uncomment all of the lines below to finish the review skill invocation.
    var response = await reviewer.InvokeAsync(context);

    string result;
    if (response.ErrorOccurred)
    {
        result = response.LastException.ToString();
    }
    else
    {
        result = response.Result;
    }

    return result;
}
```

## Test the completed code

After completing the code challenges, the team should be able to run the solution locally and walk through the following:

1. Navigate to the Adjudicators page.
    1. Select Details for a claim on the Non-Manager tab that has a Total Amount of over $500.
    2. Click on the Make Recommendation button, then click on “Ask for Recommendation”. This triggers a call into the Azure OpenAI completion model, along with the claim information and customized system prompt, orchestrated by Semantic Kernel. Read the response and close the dialog.

Does the recommendation make sense? If not, the team should review the system prompt and make adjustments as necessary. The provided system prompt works _reasonably_ well in most cases, but it may need to be tweaked to improve the results. This is where the team can get creative and experiment with different prompts to see what works best. Where they can make the most impact to the system prompt is in the `Review Rules` section. The team can add additional rules to the system prompt to improve the results. For example, if the claim is for a member that is not covered, the team can add a rule to reject the claim. If the claim is for a member that is covered, the team can add a rule to approve the claim. The team can also add rules to approve or reject claims based on the total amount of the claim.

> [!NOTE]
> At this point, the team will likely discover that Semantic Kernel does not handle mathematical operations very well. Improving this capability is a stretch goal and something they can tackle in the next challenge.
