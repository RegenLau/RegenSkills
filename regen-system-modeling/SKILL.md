---
name: regen-system-modeling
description: Turn vague or complex work into a clear system model by mapping target output, flow object, elements, nodes, connections, the minimum runnable chain, current bottleneck, next path, and reusable assets. Use when the user asks to systematize a task, clarify messy work, model a product/process/learning/project workflow, locate blockers, choose what to do next, or prevent fragmented thinking.
---

# Regen System Modeling

## Operating Rule

Use this skill to turn a messy task into a practical system map before recommending actions. Mirror the user's language. Keep the result concrete, not academic. Ask only for missing information that materially changes the model.

## Core Workflow

1. Define the target output: state what final result must exist, who uses or accepts it, and what counts as effective.
2. Identify the flow object: name what moves through the system, such as information, user intent, data, tasks, knowledge, content, money, decisions, responsibility, or state.
3. List key elements: capture the roles, data, tools, rules, permissions, resources, constraints, risks, dependencies, and acceptance standards that affect the system.
4. Map the full flow path: show how the flow object moves from source to endpoint through nodes and outputs.
5. Define nodes and connections: for each important node, state what it receives, processes, outputs, and passes downstream.
6. Extract the minimum runnable chain: reduce the system to the fewest nodes needed to run at all, usually `input -> processing -> output`.
7. Locate the current bottleneck: identify whether the issue is in the target, an element, a node, a connection, the output, or the feedback loop.
8. Choose the current path: decide the shortest useful next move under the current goal, resources, and stage, then name the reusable asset to preserve.

## Key Distinctions

- Target output defines the endpoint.
- Flow object defines the main line of movement.
- Elements define what exists in the system.
- Nodes define where processing happens.
- Connections define how nodes pass information or responsibility.
- Minimum runnable chain answers: "What is the least structure needed for this to run?"
- Current path answers: "Where should we act now to get the next valid result?"
- Feedback and assets turn a one-time analysis into a reusable system.

## Output Template

Use this structure unless the user asks for a different format:

```markdown
# [System Name] Model

## Target Output
[What the system must produce and how success is judged.]

## Flow Object
[What moves through the system.]

## Key Elements
[Roles, data, tools, rules, resources, constraints, risks, dependencies.]

## Nodes And Connections
[Source -> node -> node -> output -> feedback, with each node's input/process/output.]

## Minimum Runnable Chain
[The smallest chain that can run end to end.]

## Current Bottleneck
[The node or connection most likely limiting progress now.]

## Current Path
[The shortest useful next move and why it is first.]

## Reusable Asset
[Flowchart, checklist, decision table, PRD section, eval sheet, template, or operating rule to preserve.]
```

## Usage Notes

- Do not start with a long plan. Build the map first, then derive the plan from the map.
- Prefer a rough but complete flow over a polished incomplete diagram.
- If the user gives a specific domain, use domain terms and examples from that domain.
- If the user is learning something, model how knowledge moves from input to practice, feedback, and retained ability.
- If the user is building a product, model user intent, input, processing, output, feedback, and iteration.
