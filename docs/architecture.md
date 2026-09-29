# Architecture — Microsoft Agent Framework Multi-Agent System

## 1. Architecture Overview

This project implements a sequential multi-agent workflow using Microsoft Agent Framework and Microsoft Foundry.

The application decomposes customer-feedback analysis across three specialized AI agents:

1. **Summarizer Agent** — condenses customer feedback into a short, neutral summary.
2. **Classifier Agent** — classifies the feedback as Positive, Negative, or Feature request.
3. **Action Agent** — recommends the next action based on the information produced through the workflow.

The agents are coordinated using Microsoft Agent Framework's `SequentialBuilder`.

---

## 2. High-Level Architecture

```text
+---------------------------+
|     Customer Feedback     |
+-------------+-------------+
              |
              v
+---------------------------+
|     Summarizer Agent      |
|                           |
| Produce concise summary   |
+-------------+-------------+
              |
              v
+---------------------------+
|     Classifier Agent      |
|                           |
| Positive                  |
| Negative                  |
| Feature request           |
+-------------+-------------+
              |
              v
+---------------------------+
|       Action Agent        |
|                           |
| Recommend next action     |
+-------------+-------------+
              |
              v
+---------------------------+
|     Workflow Outputs      |
+---------------------------+
```

The participant order is explicitly defined in the workflow configuration.

---

## 3. Architectural Pattern

The application uses a **sequential multi-agent orchestration pattern**.

Instead of assigning the complete task to one agent, the workflow divides the problem into specialized stages.

Conceptually:

```text
Input
  |
  v
Specialist A
  |
  v
Specialist B
  |
  v
Specialist C
  |
  v
Workflow Result
```

For this implementation:

```text
Customer Feedback
       |
       v
Summarization
       |
       v
Classification
       |
       v
Recommended Action
```

This creates a deterministic participant sequence defined by the application.

---

## 4. Application Entry Point

The application defines an asynchronous `main()` function:

```python
async def main():
```

Execution begins with:

```python
if __name__ == "__main__":
    asyncio.run(main())
```

This provides the asynchronous runtime used by the Agent Framework workflow.

---

## 5. Configuration Layer

Environment variables are loaded using:

```python
load_dotenv()
```

Two configuration values are used:

```text
AZURE_AI_PROJECT_ENDPOINT
AZURE_AI_MODEL_DEPLOYMENT_NAME
```

They are supplied to the Foundry chat client:

```python
chat_client = FoundryChatClient(
    credential=credential,
    project_endpoint=os.getenv(
        "AZURE_AI_PROJECT_ENDPOINT"
    ),
    model=os.getenv(
        "AZURE_AI_MODEL_DEPLOYMENT_NAME"
    ),
)
```

The repository provides `.env.example` for configuration guidance.

The local `.env` file is excluded from source control.

---

## 6. Azure Authentication

The application authenticates with:

```python
credential = AzureCliCredential()
```

The credential is passed to `FoundryChatClient`.

```text
Local Python Application
          |
          v
 AzureCliCredential
          |
          v
Microsoft Foundry
```

This avoids embedding Azure credentials directly in `agents.py`.

For local development, the user authenticates separately through Azure CLI.

---

## 7. Foundry Chat Client

A single `FoundryChatClient` provides model access for all three agents.

```python
chat_client = FoundryChatClient(
    credential=credential,
    project_endpoint=os.getenv(
        "AZURE_AI_PROJECT_ENDPOINT"
    ),
    model=os.getenv(
        "AZURE_AI_MODEL_DEPLOYMENT_NAME"
    ),
)
```

The same client is used to construct each specialized agent.

Architecturally:

```text
              FoundryChatClient
               /      |      \
              /       |       \
             v        v        v
       Summarizer Classifier Action
          Agent      Agent     Agent
```

This allows multiple specialized agents to share the same configured Foundry model connection while maintaining distinct roles and instructions.

---

## 8. Agent Specialization

The system uses specialization rather than one broad instruction set.

Each agent is created with:

```python
chat_client.as_agent(...)
```

but receives a different:

- Name
- Instruction set
- Responsibility within the workflow

This separates **model connectivity** from **agent responsibility**.

---

## 9. Summarizer Agent

The first participant is:

```python
summarizer_agent = chat_client.as_agent(
    name="summarizer",
    instructions=summarizer_instructions,
)
```

Its responsibility is to summarize customer feedback in one short sentence.

The instructions specify that the summary should be:

- Short
- Neutral
- Concise

Conceptually:

```text
Detailed Feedback
       |
       v
Summarizer Agent
       |
       v
Concise Summary
```

This agent represents the first specialization boundary in the workflow.

---

## 10. Classifier Agent

The second participant is:

```python
classifier_agent = chat_client.as_agent(
    name="classifier",
    instructions=classifier_instructions,
)
```

Its instructions define three possible classifications:

```text
Positive
Negative
Feature request
```

Conceptually:

```text
Prior Workflow Context
        |
        v
Classifier Agent
        |
        v
Feedback Category
```

The classifier provides a specialized interpretation stage before the recommended-action agent executes.

---

## 11. Action Agent

The third participant is:

```python
action_agent = chat_client.as_agent(
    name="action",
    instructions=action_instructions,
)
```

Its responsibility is to suggest the next action based on the summary and classification available through the sequential workflow.

Example actions represented by its instructions include:

```text
Escalate as a high-priority bug
Log positive feedback
Add an enhancement request to the product backlog
```

Conceptually:

```text
Summary + Classification
          |
          v
      Action Agent
          |
          v
Recommended Next Action
```

---

## 12. SequentialBuilder

The multi-agent workflow is constructed using:

```python
workflow = SequentialBuilder(
    participants=[
        summarizer_agent,
        classifier_agent,
        action_agent
    ],
    output_from="all",
).build()
```

This is the central orchestration mechanism demonstrated by the project.

The participant order is:

```text
1. summarizer_agent
2. classifier_agent
3. action_agent
```

The project therefore demonstrates **sequential orchestration**.

---

## 13. Participant Ordering

The workflow explicitly defines the order in which the specialized participants are arranged.

```text
summarizer_agent
       |
       v
classifier_agent
       |
       v
action_agent
```

The application does not dynamically choose the next participant.

It supplies the participant sequence to `SequentialBuilder`.

This distinguishes the project from routing or handoff architectures where agent selection can change dynamically.

---

## 14. Orchestration Boundary

Agent specialization and orchestration are separate concepts.

### Agent Specialization

Defines **what an agent is responsible for**.

Examples:

```text
Summarizer → summarize
Classifier → classify
Action     → recommend
```

### Orchestration

Defines **how the agents are coordinated**.

For this project:

```text
SequentialBuilder
```

provides the workflow structure.

This distinction is central to the architecture.

---

## 15. Workflow Input

The sample application defines customer feedback in:

```python
feedback = """
...
"""
```

The workflow receives:

```python
result = await workflow.run(
    f"Customer feedback: {feedback}"
)
```

Conceptually:

```text
Customer Feedback String
          |
          v
   workflow.run(...)
          |
          v
Sequential Multi-Agent Workflow
```

The current project uses a predefined sample rather than interactive user input.

---

## 16. Asynchronous Workflow Execution

The workflow is executed using:

```python
await workflow.run(...)
```

inside the asynchronous `main()` function.

The runtime structure is:

```text
asyncio.run(main())
        |
        v
    async main()
        |
        v
await workflow.run(...)
```

This demonstrates asynchronous integration with the Agent Framework orchestration API.

---

## 17. Output Collection

The workflow is configured with:

```python
output_from="all"
```

After execution:

```python
outputs = result.get_outputs()
```

retrieves the workflow outputs exposed by the result.

The application then iterates through them:

```python
for response in outputs:
```

and through each response's messages:

```python
for msg in cast(
    list[Message],
    response.messages
):
```

This makes the returned workflow messages visible to the user.

---

## 18. Message Processing

The project imports:

```python
from agent_framework import Message
```

and casts returned response messages to:

```python
list[Message]
```

For each message, the application determines a display name:

```python
name = msg.author_name or (
    "assistant"
    if msg.role == "assistant"
    else "user"
)
```

The message text is then displayed.

This provides visibility into the messages returned through the orchestration result.

---

## 19. End-to-End Execution Flow

A complete execution follows this structure:

```text
1. Application starts
        |
        v
2. Environment configuration is loaded
        |
        v
3. AzureCliCredential is created
        |
        v
4. FoundryChatClient is created
        |
        v
5. Summarizer agent is created
        |
        v
6. Classifier agent is created
        |
        v
7. Action agent is created
        |
        v
8. SequentialBuilder defines workflow
        |
        v
9. Sample customer feedback is submitted
        |
        v
10. Sequential workflow executes
        |
        v
11. Workflow result is returned
        |
        v
12. Outputs are retrieved
        |
        v
13. Messages are enumerated
        |
        v
14. Agent messages are printed
```

---

## 20. Responsibility Matrix

| Component | Responsibility |
| --- | --- |
| `agents.py` | Application and workflow definition |
| `AzureCliCredential` | Azure developer authentication |
| `FoundryChatClient` | Microsoft Foundry model connectivity |
| Summarizer Agent | Condense customer feedback |
| Classifier Agent | Categorize feedback |
| Action Agent | Recommend next action |
| `SequentialBuilder` | Define sequential orchestration |
| `workflow.run()` | Execute the workflow |
| `result.get_outputs()` | Retrieve exposed workflow outputs |
| `Message` | Represent returned messages |
| `asyncio` | Run asynchronous application flow |

---

## 21. Security Boundaries

The repository separates local configuration from source code.

The repository excludes:

```text
.env
.venv/
venv/
env/
__pycache__/
```

The Python source does not embed:

- Azure passwords
- Access tokens
- API keys
- Client secrets

Authentication is delegated to `AzureCliCredential`.

Production environments should use an authentication mechanism appropriate to the hosting architecture.

---

## 22. Infrastructure Boundary

This repository does not provision Microsoft Foundry infrastructure.

Configured externally:

```text
Azure subscription
Microsoft Foundry project
Model deployment
Azure identity and permissions
```

Implemented locally:

```text
Foundry client initialization
Agent definitions
Agent instructions
Sequential workflow
Workflow execution
Output processing
```

This distinction keeps infrastructure provisioning separate from application orchestration.

---

## 23. What This Project Demonstrates

The primary engineering competency is:

> Coordinating multiple specialized AI agents through a defined sequential orchestration workflow.

The project combines:

```text
Agent Specialization
        +
Task Decomposition
        +
SequentialBuilder
        +
Microsoft Foundry
        +
Asynchronous Execution
        +
Workflow Output Processing
```

---

## 24. What This Project Does Not Implement

The project should not be presented as implementing orchestration patterns that are not present in `agents.py`.

It does not implement:

- Concurrent orchestration
- Dynamic agent routing
- Agent handoffs
- Group-chat orchestration
- Magentic orchestration
- Human approval workflows
- Persistent workflow state
- External tool integration
- MCP integration
- Infrastructure provisioning

These may be appropriate extensions for future projects.

---

## 25. Relationship to Earlier Portfolio Projects

Earlier portfolio work established competencies involving:

```text
Single Foundry agents
Agent workflows
Generative AI applications
Enterprise knowledge grounding
AI safety and guardrails
Microsoft Agent Framework
MCP-enabled agents
Foundry IQ knowledge integration
```

Project #09 introduces a different architectural capability:

```text
Multiple Specialized Agents
          +
Defined Execution Order
          +
Agent Framework Orchestration
```

This represents progression from configuring capabilities around a single agent toward coordinating multiple agent responsibilities within one workflow.

---

## 26. Engineering Progression

The portfolio progression can be represented as:

```text
Single AI Agent
      |
      v
Agent-Driven Workflow
      |
      v
AI Application
      |
      v
Knowledge-Grounded Agent
      |
      v
Guardrailed AI
      |
      v
Agent Framework
      |
      v
MCP-Enabled Agent
      |
      v
Knowledge-Enhanced MCP Agent
      |
      v
Sequential Multi-Agent System
```

Project #09 therefore introduces **multi-agent orchestration** as a distinct engineering competency.

---

## 27. Production Evolution

A production-oriented version could extend the architecture with:

```text
Customer / API Input
         |
         v
Validation Layer
         |
         v
Multi-Agent Orchestrator
   /       |        \
  v        v         v
Agent A  Agent B   Agent C
   \       |        /
         Results
            |
            v
Evaluation / Policy Layer
            |
            v
Application Response
```

Potential additions include:

- Dynamic input
- Structured outputs
- Validation
- Automated evaluation
- Retry policies
- Structured logging
- Application Insights
- Azure Monitor
- Persistent workflow state
- Managed identity
- Azure Key Vault
- Automated tests
- CI/CD
- Infrastructure as Code
- Hosted API or web application

---

## 28. Alternative Orchestration Patterns

Microsoft Agent Framework supports broader orchestration concepts beyond the pattern demonstrated here.

Future portfolio projects could explore architectures such as:

```text
Concurrent Agents
       |
       v
Result Aggregation
```

or:

```text
Agent A
   |
   +----> Agent B
   |
   +----> Agent C
```

or dynamic handoff designs.

Those patterns are intentionally outside the implementation boundary of this repository.

---

## 29. Design Principle

The central design principle demonstrated by this project is **separation of responsibility**.

Rather than using one broad agent instruction:

```text
"Analyze this feedback and decide what to do."
```

the system decomposes the task:

```text
Summarize
    |
    v
Classify
    |
    v
Recommend Action
```

This makes the orchestration structure explicit and provides a foundation for more sophisticated multi-agent architectures.

---

## 30. Summary

Project #09 demonstrates a sequential multi-agent system built with Microsoft Agent Framework and Microsoft Foundry.

Its architecture consists of:

```text
Customer Feedback
       |
       v
Summarizer Agent
       |
       v
Classifier Agent
       |
       v
Action Agent
       |
       v
Workflow Outputs
```

The agents share a Foundry-backed chat client while maintaining separate names, instructions, and responsibilities.

`SequentialBuilder` defines the orchestration structure, `workflow.run()` executes it asynchronously, and `result.get_outputs()` exposes the workflow outputs for terminal presentation.

The principal new engineering competency demonstrated by this project is **multi-agent orchestration through specialized agents operating within a defined sequential workflow**.
