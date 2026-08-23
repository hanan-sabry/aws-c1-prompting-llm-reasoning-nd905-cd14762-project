# Customer Support Chatbot with Amazon Bedrock AgentCore

## Project Overview

This project implements a customer support chatbot for a fictional online shop using **Amazon Bedrock Flows** and **Amazon Bedrock AgentCore**.

The chatbot classifies each customer message and routes it to the appropriate behavior:

* **BUG** — Collect the required bug details through the conversation and create a bug report in a database.
* **FAQ** — Answer common questions about orders, shipping, returns, and payments using the provided FAQ.
* **HUMAN** — Redirect unsupported requests to human customer support.

## Implementation Approach

### Important Note About the Project Instructions

During the implementation of this project, the project instructions were updated.

Initially, the project required using **Amazon Bedrock Agent Classic** to implement the agent behavior for the bug-reporting scenario. Since Agent Classic was deprecated/unavailable in my AWS environment, I started implementing the project using **Amazon Bedrock AgentCore** instead.

While I was progressing with the implementation, the project instructions were updated to use **AgentCore Harness**, but the new instructions described a different AgentCore architecture and integration approach from the one I had already implemented.

By the time I became aware of the updated instructions, I had already completed most of the required functionality and had a working implementation based on AgentCore. Due to the available implementation time, I decided to continue with the architecture I had already started rather than redesigning the project from the beginning.

Therefore, this README documents **my actual implementation**, which uses AgentCore Harness but integrates it with the Bedrock Flow in a different way from the approach described in the updated project instructions.

## Architecture

The overall implementation uses a **Bedrock Flow** as the orchestration layer.

```text
                         Customer Message
                                |
                                v
                       ┌─────────────────┐
                       │ Classifier      │
                       │ Prompt Node     │
                       └────────┬────────┘
                                |
                                v
                       ┌─────────────────┐
                       │ Condition Node  │
                       └───────┬─────────┘
                         /      |       \
                        /       |        \
                       v        v         v
                     BUG       FAQ      HUMAN
                      |          |          |
                      v          v          v
                   Lambda     FAQ Prompt  Human Prompt
                      |
                      v
              AgentCore Harness
                      |
                      v
              AgentCore Gateway
                      |
                      v
           create-bug-report Tool
                      |
                      v
                  Database
```

## 1. Customer Message Classification

The first step of the Bedrock Flow uses a **Prompt node** to classify the customer's message into one of three categories:

* `BUG`
* `FAQ`
* `HUMAN`

The classifier prompt determines the customer's intent and returns the appropriate category.

The classifier prompt is included in the project files.

## 2. Conditional Routing

The classification result is passed to a **Condition node**, which routes the conversation to the appropriate path.

This keeps the main Flow responsible for high-level orchestration while allowing each type of request to have its own dedicated behavior.

## 3. BUG Path — AgentCore Harness

The BUG path is where the agent-based behavior is implemented.

Instead of using the originally required **Agent Classic**, I implemented an **Amazon Bedrock AgentCore Harness**.

The Harness is configured to work with an **AgentCore Gateway**, and the Gateway exposes the `create-bug-report` tool as a configured target.

The tool is responsible for creating the bug report in the database.

The important note: I had already established this architecture before the updated instructions became available, so I continued with the working implementation.

## 4. Invoking AgentCore from the Bedrock Flow

To connect the Bedrock Flow with the AgentCore Harness, I used a **Lambda node as an integration/adapter layer**.

The Lambda node invokes the AgentCore Harness and allows the Flow to continue with the bug-reporting workflow.

This results in the following responsibility separation:

| Component             | Responsibility                                             |
| --------------------- | ---------------------------------------------------------- |
| **Bedrock Flow**      | Orchestrates the conversation and routes customer requests |
| **Classifier Prompt** | Classifies the customer message                            |
| **Condition Node**    | Selects the appropriate path                               |
| **Lambda**            | Integrates the Flow with the AgentCore Harness             |
| **AgentCore Harness** | Provides the agent-based behavior                          |
| **AgentCore Gateway** | Exposes the bug-reporting tool                             |
| **create-bug-report** | Creates the bug record in the database                     |

## 5. FAQ Path

Messages classified as `FAQ` are routed to a dedicated **FAQ Prompt node**.

The supported FAQ information is injected into the prompt, allowing the model to answer questions related to:

* Orders
* Shipping
* Returns
* Payments

The FAQ prompt is included in the project files.

## 6. HUMAN Path

Messages classified as `HUMAN` are routed to a dedicated prompt.

This prompt politely informs the customer that the request requires human assistance and redirects them to the provided customer support phone line.

The HUMAN prompt is included in the project files.

## Included Project Files

The project includes the prompts used by the implementation:

* **agentcore_harness_prompt** — defines the behavior of the bug-reporting agent.
* **classifier_prompt** — classifies messages into `BUG`, `FAQ`, or `HUMAN`.
* **faq_prompt** — handles supported FAQ questions.
* **human_prompt** — redirects unsupported requests to human support.

I also included screenshots documenting the AWS Console configuration of:

* The **AgentCore Harness**
* The **AgentCore Gateway**
* The Gateway target configuration for the `create-bug-report` tool

## Files Not Included

Because my implementation follows the AgentCore architecture that I had already started before the updated project instructions were available, some files mentioned in the project instructions are not part of my implementation.

### `system_prompt.txt`

I did not use a separate `system_prompt.txt`.

Instead, the relevant behavior is defined through the AgentCore Harness prompt and the other prompts included in the project.

### `agentcore_config.json`

I did not use this configuration file because I configured the AgentCore Harness through the **AWS Console**.

Screenshots of the actual Harness configuration are included instead.

### `create_harness.py`

I did not use a Python script to create the Harness.

The Harness was configured directly through the AWS Console, and screenshots of the configuration are provided.

### `setup_gateway.py`

I did not use this script to configure the Gateway.

The AgentCore Gateway and its `create-bug-report` target were configured through the AWS Console. Screenshots of this configuration are included.

## Summary

The final implementation provides the required customer-support behaviors using **Amazon Bedrock Flows and AgentCore**:

* Customer messages are classified into **BUG, FAQ, or HUMAN**.
* The Flow routes each request to the appropriate path.
* FAQ requests are answered using embedded FAQ information.
* Other requests are redirected to human support.
* Bug requests use an **AgentCore Harness** integrated with an **AgentCore Gateway** and the `create-bug-report` tool to create the bug record in the database.

The main implementation decision was driven by the transition in the project requirements. I initially moved from the unavailable/deprecated Agent Classic approach to AgentCore, and when the instructions were subsequently updated to provide a more detailed AgentCore Harness implementation using a different architecture, I continued with the working AgentCore architecture I had already developed.

This README therefore documents the **actual implementation submitted**, rather than presenting an architecture that was not used in the project.
