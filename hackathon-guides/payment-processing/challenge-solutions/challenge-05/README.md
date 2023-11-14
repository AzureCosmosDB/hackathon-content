# Solution for Challenge 05

> [!NOTE]
> There is currently not a solution for this challenge, but only hints and tips to help you coach your team. As a coach, you should allow the team to choose their own path, as long as they meet the success criteria. It is **not** expected that a coach will have deep expertise in all possible paths, nor expert level command of all possible syntax. Coaches can lean on each other for help if a team chooses a less familiar path. If no coach has experience with the team’s chosen path, this is a growth opportunity for the coach to join in and learn **with** the team.
> 
> The expectation is, as we deliver this hackathon, we will add solutions for the most common paths. If you have a solution you would like to share, please submit a pull request.

## Challenge description

You may have found that sometimes the SemanticFunction works and provides an accurate result, and sometimes it appears to calculate things incorrectly or have flaws in its logic.

In this challenge you improve the the capabilities in two ways:

- Improve the prompt text to reduce any dependency on the model's parametric knowledge (this is the knowledge the model has learned during pre-training) so that it only uses the data you supply and the user query in the processing.
- Improve the handling of numbers. Sometimes numbers are better handled with native code. You will need to improve SemanticFunction by leveraging the SequentialPlanner that uses a Semantic Kernel plugin for numeric operations in addition to your SemanticFunction.

## Challenge

Your team must:

1. Improve the SemanticFunction to only use contextual knowledge that you provide it.
2. Improve the handling of numbers.

### Success Criteria

To complete this challenge successfully, you must:

- Demonstrate that the SemanticFunction only answers questions about the data context you provide and tells you when it doesn't have enough information to answer the question.
- Ask questions that require numeric operations and demonstrate that the SequentialPlanner uses your plugin to answer the question.

### Hints and tips

- Use system prompts to instruct the model to only answer questions about the context you provide.
- Look at sample Semantic Kernel plugins to get an idea of how to create a new plugin: https://github.com/microsoft/semantic-kernel/tree/main/dotnet/src/Plugins/Plugins.Core
- The existing `MathPlugin` (https://github.com/microsoft/semantic-kernel/blob/main/dotnet/src/Plugins/Plugins.Core/MathPlugin.cs) could work with little modification.
- Update the `AnalyticsEngine` class to use the new plugin. You could update the system prompt (`skPrompt`) to use `{{math.Add}}` on the `Transaction.amount`. Be aware that the `amount` value is always positive. You need to instruct the agent to evaluate the `type` property to add if it is a "deposit", and subtract if it is a "debit".

### Resources

- [Automatically orchestrate AI with planner](https://learn.microsoft.com/semantic-kernel/ai-orchestration/planner?tabs%253DCsharp)
- [Creating semantic plugins](https://learn.microsoft.com/semantic-kernel/ai-orchestration/plugins/?tabs=Csharp)
- [Semantic Kernel native functions](https://learn.microsoft.com/semantic-kernel/ai-orchestration/native-functions)
- [Prompt engineering techniques](https://learn.microsoft.com/azure/cognitive-services/openai/concepts/advanced-prompt-engineering?pivots%253Dprogramming-language-chat-completions)
