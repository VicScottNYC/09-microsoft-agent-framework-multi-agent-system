# Microsoft Agent Framework Multi-Agent System

A sequential multi-agent AI system built with **Microsoft Agent Framework** and **Microsoft Foundry**. The application decomposes customer-feedback analysis across three specialized AI agents responsible for summarization, classification, and recommended action.

## Project Overview

This project demonstrates how multiple specialized AI agents can collaborate through a deterministic orchestration workflow.

Rather than assigning the entire task to a single general-purpose agent, the application separates the workload across three agents:

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
Collected Workflow Outputs
```

The workflow is constructed using Microsoft Agent Framework's `SequentialBuilder`.

Each agent has a specific responsibility and participates in a defined execution sequence.

The project demonstrates:

- Microsoft Agent Framework
- Microsoft Foundry model integration
- Specialized agent design
- Sequential multi-agent orchestration
- Agent-specific instructions
- Workflow construction
- Ordered agent execution
- Workflow output collection
- Asynchronous Python execution
- Azure CLI authentication
- Environment-based configuration

---

## Multi-Agent Architecture

The system contains three specialized agents.

### 1. Summarizer Agent

The summarizer condenses the original customer feedback into one short, neutral sentence.

Example intent:

```text
Customer provides detailed product feedback
                |
                v
        Summarizer Agent
                |
                v
      Concise feedback summary
```

Its instructions emphasize neutrality and brevity.

---

### 2. Classifier Agent

The classifier categorizes the feedback into one of three classes:

```text
Positive
Negative
Feature request
```

This creates a structured interpretation of the feedback for downstream processing.

---

### 3. Action Agent

The final agent recommends the next action based on the information produced through the sequential workflow.

Example actions represented in the agent instructions include:

```text
Escalate a high-priority bug
Record positive feedback
Add an enhancement request to the product backlog
```

The result is a pipeline in which specialized agents contribute different stages of analysis.

---

## Sequential Orchestration

The agents are orchestrated with:

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

The participant order defines the sequential workflow:

```text
summarizer_agent
       |
       v
classifier_agent
       |
       v
action_agent
```

This project specifically demonstrates **sequential orchestration**.

It does not claim to implement concurrent, handoff, group-chat, or Magentic orchestration.

---

## Why Use Multiple Agents?

A single AI agent could potentially perform all three operations.

The architectural purpose of this project is different: it demonstrates **task decomposition through agent specialization**.

Each agent receives its own role and instructions.

```text
                    +----------------------+
                    |  Customer Feedback   |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |  Summarizer Agent    |
                    |  Condense feedback   |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |  Classifier Agent    |
                    |  Categorize feedback |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |    Action Agent      |
                    | Recommend next step  |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Workflow Outputs     |
                    +----------------------+
```

This pattern provides a foundation for larger agent systems in which responsibilities are distributed across specialized components.

---

## Microsoft Foundry Integration

The application creates a Foundry-backed chat client:

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

The client provides the connection between Microsoft Agent Framework and the configured model deployment in Microsoft Foundry.

---

## Azure Authentication

Authentication uses:

```python
credential = AzureCliCredential()
```

This enables the local application to authenticate using the developer's Azure CLI session without embedding Azure credentials in the Python source.

For local development, authenticate before running the application:

```bash
az login
```

No Azure passwords, access tokens, or API keys should be stored in the repository.

---

## Agent Creation

Each specialized agent is created from the same Foundry-backed chat client while receiving its own name and instructions.

### Summarizer

```python
summarizer_agent = chat_client.as_agent(
    name="summarizer",
    instructions=summarizer_instructions,
)
```

### Classifier

```python
classifier_agent = chat_client.as_agent(
    name="classifier",
    instructions=classifier_instructions,
)
```

### Recommended Action

```python
action_agent = chat_client.as_agent(
    name="action",
    instructions=action_instructions,
)
```

This demonstrates a reusable model-client architecture in which multiple specialized agents can be created from a common Foundry connection.

---

## Workflow Execution

The orchestration is executed asynchronously:

```python
result = await workflow.run(
    f"Customer feedback: {feedback}"
)
```

The application entry point uses:

```python
asyncio.run(main())
```

This establishes an asynchronous execution model for the Agent Framework workflow.

---

## Workflow Outputs

The workflow is configured with:

```python
output_from="all"
```

After execution, outputs are retrieved with:

```python
outputs = result.get_outputs()
```

The application iterates through the returned responses and their messages:

```python
for response in outputs:
    for msg in cast(list[Message], response.messages):
        ...
```

Each message is displayed with its corresponding author name when available.

This makes the contributions produced during the workflow visible in the terminal.

---

## Example Use Case

The sample application analyzes customer feedback about a product dashboard.

The supplied feedback describes:

- Regular dashboard usage
- General satisfaction with the product
- Discomfort caused by the bright interface at night
- A request for dark mode

The agents then process that feedback through their specialized roles.

Conceptually:

```text
Detailed Customer Feedback
          |
          v
+---------------------+
| Summarizer          |
| Short neutral       |
| summary             |
+----------+----------+
           |
           v
+---------------------+
| Classifier          |
| Positive / Negative |
| / Feature request   |
+----------+----------+
           |
           v
+---------------------+
| Action              |
| Recommended next    |
| step                |
+----------+----------+
           |
           v
   Workflow Outputs
```

Actual model-generated wording may vary between executions.

---

## Project Structure

```text
09-microsoft-agent-framework-multi-agent-system/
├── .env.example
├── .gitignore
├── README.md
├── agents.py
├── requirements.txt
└── docs/
    └── architecture.md
```

---

## Technologies

- Python
- Microsoft Agent Framework
- Microsoft Foundry
- Agent Framework Foundry integration
- Agent Framework Orchestrations
- Azure Identity
- `AzureCliCredential`
- `SequentialBuilder`
- `asyncio`
- `python-dotenv`

---

## Dependencies

The project uses:

```text
python-dotenv
azure-identity==1.25.3
agent-framework-core==1.12.1
agent-framework-foundry==1.10.3
agent-framework-orchestrations==1.0.1
```

---

## Prerequisites

Before running the application, you need:

- Python 3
- An Azure subscription
- Azure CLI
- Access to a Microsoft Foundry project
- A model deployment available to the project
- Appropriate Azure permissions

The Microsoft Foundry project and model deployment are configured externally and are not provisioned by this repository.

---

## Installation

Clone the repository:

```bash
git clone <repository-url>
cd 09-microsoft-agent-framework-multi-agent-system
```

Create a virtual environment:

```bash
python3 -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

---

## Configuration

Create a local environment file from the example:

```bash
cp .env.example .env
```

Configure:

```text
AZURE_AI_PROJECT_ENDPOINT=<your-foundry-project-endpoint>
AZURE_AI_MODEL_DEPLOYMENT_NAME=<your-model-deployment-name>
```

The `.env` file is excluded from Git and should never be committed.

---

## Authentication

Authenticate through Azure CLI:

```bash
az login
```

The application then uses:

```python
AzureCliCredential()
```

to obtain the Azure identity required by the Foundry client.

---

## Running the Application

Run:

```bash
python agents.py
```

The application will:

1. Authenticate to Azure.
2. Connect to the configured Microsoft Foundry project/model.
3. Create the three specialized agents.
4. Build the sequential workflow.
5. Submit the sample customer feedback.
6. Execute the multi-agent orchestration.
7. Collect the workflow outputs.
8. Display the returned agent messages.

---

## Security Considerations

The repository intentionally excludes:

- `.env`
- Azure credentials
- Access tokens
- Local virtual environments
- Python cache files

Configuration is loaded through environment variables.

Authentication uses Azure CLI credentials rather than credentials embedded directly in source code.

Production deployments should use an identity and secret-management strategy appropriate for the hosting environment.

---

## Engineering Skills Demonstrated

This project demonstrates:

- Multi-agent system design
- Agent specialization
- Task decomposition
- Sequential agent orchestration
- Microsoft Agent Framework
- Microsoft Foundry integration
- `SequentialBuilder`
- Workflow construction
- Agent instruction design
- Workflow result processing
- Azure authentication
- Asynchronous Python
- Environment-based configuration

---

## Implementation Boundaries

This repository implements:

```text
Three specialized agents
Agent-specific instructions
Foundry chat-client integration
Sequential workflow construction
Asynchronous workflow execution
Workflow output collection
Terminal output presentation
```

The repository does not provision:

```text
Microsoft Foundry infrastructure
Azure subscriptions
Model deployments
Azure identity infrastructure
```

Those resources are configured externally.

The project demonstrates **sequential multi-agent orchestration** and should not be interpreted as an implementation of every orchestration pattern supported by Microsoft Agent Framework.

---

## Portfolio Progression

This is **Project #09** in a Microsoft Foundry and Azure AI engineering portfolio.

Earlier projects established:

- Foundational Microsoft Foundry agents
- Agent-driven workflows
- Generative AI applications
- Enterprise knowledge grounding
- Generative AI guardrails
- Microsoft Agent Framework
- MCP-enabled agent capabilities
- Foundry IQ and MCP knowledge integration

Project #09 introduces a distinct engineering competency:

> **Multiple specialized AI agents coordinated through a defined orchestration workflow.**

The architectural progression is:

```text
Single Agent
     |
     v
Agent + Tools
     |
     v
Agent + Enterprise Knowledge
     |
     v
Multiple Specialized Agents
     |
     v
Multi-Agent Orchestration
```

---

## Future Enhancements

Potential extensions include:

- Dynamic customer-feedback input
- Structured output models
- Automated workflow evaluation
- Concurrent orchestration
- Agent handoff patterns
- Group-chat orchestration
- Magentic orchestration
- Human-in-the-loop review
- Persistent workflow state
- Application Insights
- Structured logging
- Automated testing
- CI/CD with GitHub Actions
- Azure-hosted deployment
- Infrastructure as Code
- Production identity using managed identity

---

## Training Context

The initial implementation was developed while completing Microsoft's exercise **Develop a multi-agent solution with Microsoft Agent Framework**.

The portfolio version focuses on the engineering architecture and competencies demonstrated by the completed application.

---

## License

This repository is intended for educational and portfolio demonstration purposes.
