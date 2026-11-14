---
name: running-data-flows
description: "Execution skill. Use when the user asks to run, trigger, launch or re-run a Data Flow, or to reload, resync or refresh data that an existing Data Flow already loads (imports and data copies), or asks which Data Flows an Application has, what they do, or which one to use. Covers finding the Data Flow with list_data_flows and triggering it with run_data_flow, including the approval, asynchronous execution and error handling. Do NOT use to create or edit a Data Flow or its steps, to import a CSV file (use Pigment's data import UI), or to copy metric data by creating a new copy config (see `skill:copying-metric-data`)."
metadata:
  base_directory: /skills/running-data-flows
  includes:
    - "SKILL.md"
---

# Running Data Flows

A Data Flow is an ordered sequence of Data Actions (imports and data copies) that run as a whole, in order. `tool:list_data_flows` finds them and `tool:run_data_flow` triggers one.

You can only **run** existing Data Flows. You cannot create or edit a Data Flow, change its steps, upload a file for a step, or read the status of an execution.

## Step 1: Find the Data Flow

Call `tool:list_data_flows` with the `applicationId` of the Application that owns the Data Flow. Get the ID from `tool:get_applications` if you do not have it. For each Data Flow it returns:

- `dataFlowId`: the ID to pass to `tool:run_data_flow`.
- `name` and `description` (the description can be null): use them to match the user's request.
- `stepCount`: the number of steps. Zero means there is nothing to run.

Results are sorted by name. An empty list means the Application has no Data Flow: tell the user, and do not look for a workaround.

Pick the Data Flow that matches the request. If several plausibly match, or none clearly does, show the candidates (name, description, number of steps) and ask the user which one to run. Never guess between two Data Flows: running the wrong one overwrites data.

The tool fails with `DataFlowsReadForbidden` when the user cannot run or configure Data Flows in that Application. Tell them they lack that permission instead of retrying.

## Step 2: Run it

Call `tool:run_data_flow` with only the `dataFlowId`. There is no `applicationId` argument: the Application is derived from the Data Flow.

- **It modifies data.** All the steps run in order and write to the Application. The user is always asked to approve before the call runs, even when other tools are auto-approved, so state in your message which Data Flow you are about to run and what it is for.
- **It is not idempotent.** Each call starts a new execution. Never call it twice to "make sure": that runs the Data Flow twice. If a call is rejected by the user, do not re-issue it.
- **It does not wait.** The call returns as soon as the execution is triggered, with a `dataFlowExecutionId`. A success only means the execution was **started**, not that it finished or that it succeeded.

## Step 3: Report and verify

Tell the user the Data Flow was triggered, and that it runs in the background. Do not say that the data is updated.

There is no tool to read the execution status. If the user needs the result, either ask them to check the execution status in the Data Flow's history in the Application, or look at the target data with the search and data-reading tools you have, after a short wait. Data that has not changed yet may mean the execution is still running, so do not conclude it failed, and never re-run it to find out. If you cannot confirm the result, say so.

When the Data Flow is only one part of a bigger task (for example refreshing data before building a View on it), do not build on the new data before the user confirms the execution has finished.

## Errors

| Error reason | Meaning | What to do |
| --- | --- | --- |
| `DataFlowNotFound` | The ID does not exist, or was deleted. | Call `tool:list_data_flows` again and use an ID it returns. |
| `DataFlowRunForbidden` | The user cannot run Data Flows in the owning Application. | Tell the user. They need that permission from an Application admin. |
| `DataFlowHasNoStep` | The Data Flow has no step. | Tell the user it is empty and needs steps added by someone who can configure it. |
| `DataFlowStepCannotBeRun` | A step cannot run from here. Typical causes: a CSV import step that needs an uploaded file, or a target Application that is read-only. The message names the first failing step. | Relay the message. Ask the user to run it from the Pigment UI, where they can supply the file. |
| `DataFlowTooManyConcurrentExecutions` | Too many executions of this Data Flow are already running. | Do not retry in a loop. Tell the user an execution is already in progress and to try again later. |

Other errors (the user is not authenticated, the service is unavailable) are not specific to Data Flows: report them and stop.
