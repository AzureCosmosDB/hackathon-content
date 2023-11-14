# Solution for Challenge 05

> [!NOTE]
> There is currently not a solution for this challenge, but only hints and tips to help you coach your team. As a coach, you should allow the team to choose their own path, as long as they meet the success criteria. It is **not** expected that a coach will have deep expertise in all possible paths, nor expert level command of all possible syntax. Coaches can lean on each other for help if a team chooses a less familiar path. If no coach has experience with the team’s chosen path, this is a growth opportunity for the coach to join in and learn **with** the team.
> 
> The expectation is, as we deliver this hackathon, we will add solutions for the most common paths. If you have a solution you would like to share, please submit a pull request.

## Challenge description

Create a Semantic Kernel plugin to automatically review, approve, deny, or forward the claim to a manager, replacing the hardcoded rule logic currently used by the solution.

## Challenge

Your team must:

1. Create a new Semantic Kernel plugin to automatically review, approve, deny or forward the claim to a manager.
2. Modify the `RulesEngine` class to implement the new plugin.
3. Generate new claims to test and verify the new plugin's capability to make claim decisions.

### Success criteria

To complete this challenge successfully, you must:

- Have a working implementation of the new Semantic Kernel plugin that can make claim decisions.
- Integrate the plugin into your solution.
- Successfully generate new claims and verify that the plugin is making claim decisions.

### Hints and tips

- Look at sample Semantic Kernel plugins to get an idea of how to create a new plugin: https://github.com/microsoft/semantic-kernel/tree/main/dotnet/src/Plugins/Plugins.Core
- Create a new Visual Studio project for the plugin. This gives you a place to write and test your code before integrating it into the solution. Here's an example: https://github.com/microsoft/semantic-kernel/tree/main/dotnet/src/Plugins/Plugins.Document
- Update the `RulesEngine` class to use the new plugin. The example plugins have sample code for using it within the XML documentation on the classes.

### Resources

- [Automatically orchestrate AI with planner](https://learn.microsoft.com/semantic-kernel/ai-orchestration/planner?tabs%253DCsharp)
- [Creating semantic plugins](https://learn.microsoft.com/semantic-kernel/ai-orchestration/plugins/?tabs=Csharp)
- [Creating semantic functions](https://learn.microsoft.com/semantic-kernel/ai-orchestration/semantic-functions?tabs%253DCsharp)
- [Semantic Kernel native functions](https://learn.microsoft.com/semantic-kernel/ai-orchestration/native-functions)
- [Prompt engineering techniques](https://learn.microsoft.com/azure/cognitive-services/openai/concepts/advanced-prompt-engineering?pivots%253Dprogramming-language-chat-completions)
