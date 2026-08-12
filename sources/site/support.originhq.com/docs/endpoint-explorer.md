# Source: https://support.originhq.com/docs/endpoint-explorer

The **Overview** dashboard is the main observability view in the Origin console. It provides a fleet-wide summary of AI activity across all reporting endpoints, broken down by agent and model over a selectable time range.

![](https://files.readme.io/633b70663eb81bff73b79de8dcb3b5c3998caf2135997cf842aeb73dfc8b814e-Origin-endpoint-explorer.png)

## 

Left panel

- **Date range picker** — scopes the entire view to a specific window. Click to select a preset or enter a custom range.
- **Overview** — navigates to the fleet-wide summary shown above.
- **Agents** — lists every AI agent detected in the selected period, with a total count shown next to the section header. Each entry includes the agent name, an activity sparkline for the selected range, and a total request count. Use **Filter agents** to search by name and **Sort** to reorder (e.g. Most active). Agents that cannot be identified are grouped under **Unknown Agent**.

## 

Right panel — Overview

The Overview panel shows three headline metrics for the selected time range:

| Metric | Description |
| --- | --- |
| **Requests** | Total number of AI API requests captured across all endpoints |
| **Endpoints** | Number of distinct endpoints that reported activity |
| **Sessions** | Number of distinct user sessions observed |

### 

Activity Surface

The **Overview activity** chart displays fleet activity over time. Use the time-range buttons (**1D, 5D, 1M, 1Q, 1Y**) in the top-right of the chart to change the visible window. Switch between tabs to view different dimensions of activity:

- **Active endpoints** — count of endpoints making AI requests over time
- **Active sessions** — session volume over time
- **Prompt volume** — number of prompts sent across all providers
- **Token volume** — total tokens exchanged; hover over any point to see a breakdown by specific model version for that interval
- **Cluster swimlane** — activity grouped by behavioral cluster

## 

Bottom panel

- **Trace Log** — a chronological record of individual AI requests captured by the fleet
- **Execution Log** — a record of agent-level execution events

 

Copy Page