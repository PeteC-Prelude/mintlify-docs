# Source: https://support.originhq.com/docs/activity-clustering

**Activity Clustering** groups AI activity into semantically related clusters so you can see at a glance what your organization is actually using AI for. Clusters are named automatically from the prompts inside them. The **Contrast** view maps agents against clusters in a matrix, making it easy to spot which agents are driving which types of activity.

![](https://files.readme.io/cce3399bfc67ddcdc285a282a1c736900999f25eea074eb8f989e6ea38f5de7e-origin-organizational-clustering.png)

## 

Left panel — Filters

The left panel scopes what appears in the matrix. It shares the same controls as the Overview dashboard:

- **Date range picker** — sets the time window for the activity being analyzed
- **Agents** — lists all detected agents with sparklines and request counts. Use **Filter agents** to search by name and **Sort** to reorder. Selecting an agent here filters the matrix to that agent's activity.

## 

Right panel — Contrast matrix

The right panel shows the **Organization** context with a **Contrast** view. The matrix rows and columns are configurable:

- **ROWS** — set to **Agents** by default; each row is an AI agent detected in the fleet
- **COLS** — set to **Clusters** by default; each column is an auto-named activity cluster

Each cell in the matrix represents activity for that agent/cluster combination. The dot size and color indicate relative activity level:

- **Large dark dot** — that agent's activity in this cluster is above the baseline rate for the organization
- **Small faded dot** — activity at or below the baseline rate

Hover over any cell to inspect the detail for that agent and cluster combination.

The cluster column headers are automatically generated names derived from the prompts in each cluster (e.g. "Rust Code In...", "Security Review...", "gRPC Endpoi..."). Truncated names can be read in full by hovering the column header.

 

Copy Page