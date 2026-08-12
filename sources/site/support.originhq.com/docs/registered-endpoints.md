# Source: https://support.originhq.com/docs/registered-endpoints

The **Endpoint Inventory** is the canonical list of every endpoint where the Origin agent has reported in. It is accessible from the Settings navigation under **Endpoint Inventory**. Each row represents one endpoint and shows key details about the host, the registered user, the installed AI agents, and the Origin agent itself.

![](https://files.readme.io/054b2a10a53fcf648b086c9f86ec0cf5310ab9ccb674f2c31e12ae6bbf00e44a-Origin-Registered_Endpoints-June1.png)

## 

Search and filtering

The **Search** box at the top filters the table in real time across hostname, agent version, registered user, model, OS, and other fields. The counter below the search box shows how many endpoints are currently displayed versus the total (e.g. _200 shown 270 total_).

## 

Column management

Click **COLUMNS** in the top-right corner to toggle which columns are visible. The button shows how many columns are currently active out of the total available (e.g. _6/19_). Use this to focus the view on the fields most relevant to your task.

## 

Default columns

| Column | Description |
| --- | --- |
| **Hostname** | The host's reported computer name |
| **Last Seen** | Date and time the endpoint last checked in; the table sorts by this column by default |
| **Operating System** | macOS or Windows |
| **Registered User** | The user account associated with the endpoint at last check-in |
| **Agent Version** | The version of the Origin agent running on this host |
| **Installed AI** | AI applications and tools detected on the host |

Additional columns are available via the **COLUMNS** toggle, including hardware details (serial, manufacturer, model, processor, memory, drive size), OS version and edition, and agent integrity fields (executable SHA256, watchdog version).

Copy Page