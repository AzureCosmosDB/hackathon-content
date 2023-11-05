# Coaches Guide - Build Modern AI Apps Hackathon

Coach's guide for delivering the [Build Modern AI Apps hackathon](https://github.com/Azure/Build-Modern-AI-Apps-Hackathon).

## An introduction for coaches

A hackathon is a hands-on experience in which participants work in team to solve a
series of challenges based on the requirements and resources provided. It
is *not* designed to be delivered as a traditional training session or hands-on
lab.

Your role as a coach is to encourage your team to work together to solve the
challenges for themselves. You may provide critical assessment of their ideas
and suggest potential avenues of exploration when they get stuck; but you should
*not* provide solutions to the challenges. The outcome that customers value most
from the hackathon experience is the satisfaction of having solved the challenges for
themselves, and the in-depth learning that comes from that experience.

So what *is* expected of a coach?

- Facilitating intra- (and inter-) team work to solve the challenges. Lead
    team discussions, being sure to include all team members. Act as a "sounding
    board" for team brainstorming while helping the team focused on the
    challenges and their success criteria.

- Unblocking technical issues that are not directly related to the challenges
    (for example, command line syntax or network authentication issues).

- Explaining underlying concepts where necessary. Participants can often find
    the code they need in documentation or on sites like StackOverflow; but they
    may have difficulty understanding what it is that they're actually doing.

- Managing team sentiment and morale – the challenges are designed to be
    challenging; and as a result, some participants may experience frustration
    at times during the event. Coaches need to be sensitive to the mood of
    the team and help steer them towards a breakthrough by asking leading
    questions, proposing alternative avenues of exploration, or just suggesting
    a coffee break.

- Validating challenge solutions against the specified criteria, and approving
    team progress through the challenges; being sure to celebrate the team’s
    successes along the way!

> [!NOTE]
> For more coach guidance, see the [Coaching Guide](/coaching-guide/README.md).

## Preparing to Coach the Build Modern AI Apps Hackathon

Before coaching the hackathon, you should ensure you are
fully prepared.

### 1. Build Foundational Knowledge

Coaches for the Build & Modernize AI Apps hackathon require knowledge about Azure Cosmos DB, vector databases, RAG patterns, event sourcing, embedding, Azure OpenAI, application 
development, and orchestration using Semantic Kernel. There is a lot to cover!

You should read and understand the following services and concepts:

Services:

- [Azure Cosmos DB](https://learn.microsoft.com/azure/cosmos-db/)
- [Azure OpenAI](https://learn.microsoft.com/azure/cognitive-services/openai/overview)
- [Azure Cognitive Search](https://learn.microsoft.com/azure/search/)

Concepts:
- [Embeddings](https://learn.microsoft.com/azure/cognitive-services/openai/concepts/understand-embeddings)
- [Prompt engineering](https://learn.microsoft.com/semantic-kernel/overview/)
- [Effective system prompts](https://learn.microsoft.com/azure/cognitive-services/openai/concepts/system-message)

Most importantly, you need to fully understand the solution accelerator, what it does and how it works.

- [Watch the Deep Dive Video](https://aka.ms/vsaia.deepdive) You may want to watch this video up through the installation section, then follow along for the remainder of the video as we go through the source code for this solution.
- [Clone and get the solution setup and working](https://github.com/Azure/Vector-Search-AI-Assistant/tree/cognitive-search-vector) 

There is also content which you can use to redeliver the deep dive video yourself including:
- [Deep Dive PowerPoint Presentation](./deep-dive/Build_Modern_AI_Apps_Solution_Deep_Dive.pptx) and includes talk track
- [Deep Dive Demo Script](./deep-dive/BuildModernAIAppsDeepDiveScript.docx) includes EVERYTHING else you need to deliver this session.

### 2. Complete the Hackathon Challenges

Ideally, you should attend the hackathon as a participant before coaching
it. If this is not possible, you should complete all the challenges on your
own. You should try to complete the challenges using only the instructions
that the participants will have, but you can also use the notes in
the rest of this guide for more detailed information and context that may
help you coach your team during the hackathon event itself.

> **Note**: If a **code** folder is provided for a challenge,
> these solutions were used to validate the challenges.
> Obviously, you might find this useful; but we encourage you to try to
> *solve the challenges for yourself before referring to the solutions*.
>
> The **code** folder includes a solution for paths that we anticipate
> being the most common choices. As a coach, you must allow a team to choose
> alternative ways to complete a challenge, if the success criteria is met. It is **not** expected
> that a coach will have deep expertise in all possible paths, nor expert
> level command of all possible syntax. Coaches can lean on each other for
> help if a team chooses a less familiar path. If no coach has experience with
> the team’s chosen path, this is a growth opportunity for the coach to join
> in and learn **with** the team.

### 3. Prepare to Support Your Team

The guidance in the rest of this document incorporates insights from the
content authors that describe the learning objectives for the challenges and
how they are intended to be used. Over time, this section will grow with
hints and tips from other coaches who have successfully helped customers as they
work through the challenges. Be sure to read through these notes, as they will help you
ensure your team has a positive and successful hackathon experience.

## Challenge Guidance

The rest of this content in this folder provides guidance for helping your team as they work
through the challenges.

### General Expectations

The hackathon consists of a series of challenges that reflect a logical order for
implementing a modern cloud-native application for enhancing an existing retail-based
application with an "intelligent agent" that allows users to ask questions on vectorized
product, customer, and sales order data stored in a database. If the learner worked
for an organization that tasked them to build a POC of a chat interface that provides
this functionality, they would likely follow a similar path.
