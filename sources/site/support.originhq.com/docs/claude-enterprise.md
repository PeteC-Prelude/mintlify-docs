# Source: https://support.originhq.com/docs/claude-enterprise

Origin connects to Claude Enterprise through Anthropic's Compliance API. It pulls your organization's Claude activity directly from Anthropic and lands it in the same context graph as everything Origin observes on the endpoint.

---

## 

What it adds

Origin's agent already captures Claude where it runs on an instrumented endpoint, including Claude Code, the desktop app, Cowork, and browser-based agents. The Compliance API covers the rest of the organization: claude.ai and Claude Desktop usage on endpoints that are not running the agent, plus organization-level activity such as sign-ins, admin actions, and configuration changes. Both feed the same record of AI work.

With the key in place, Origin pulls Claude chats and messages, the files, projects, and artifacts attached to them, and activity feed events. Each chat is attributed to the user who ran it and threaded into the same traces, analytics, and inventory as the rest of your AI activity, so Claude usage can be investigated alongside the endpoint record.

---

## 

How it works

The connector runs on its own once the key is saved. It reads Claude's activity feed about once a minute, picks up the chats that changed, and pulls their new messages. The first sync backfills roughly the last 30 days. There is nothing to export and no schedule to manage.

The key is read-only. Origin reads from your Claude organization and does not write to it or delete anything.

---

## 

Before you start

- A Claude Enterprise plan with the Compliance API available. It is generally available on Claude Enterprise, except for Public Sector organizations.
- The Primary Owner of your Claude organization, who can enable the Compliance API and create the key.
- An Origin admin, who can save the key.

---

## 

Step 1: Enable the Compliance API in Claude

The Primary Owner does this once.

1. Sign in to [claude.ai](https://claude.ai) as the Primary Owner.
2. Open **Organization settings > API** ([claude.ai/admin-settings/api-access](https://claude.ai/admin-settings/api-access)).
3. Under **Compliance API**, click **Enable**.

---

## 

Step 2: Create a Compliance Access Key

1. In claude.ai, go to **Settings > Data and Privacy > Compliance access keys**.
2. Click **Create key** and name it, for example `Origin`.
3. Select these three scopes:
 - `read:compliance_activities`
 - `read:compliance_user_data`
 - `read:compliance_org_data`
4. Click **Create** and copy the key. It begins with `sk-ant-api01-`, and Claude shows it once, so store it somewhere safe.

Origin uses all three scopes: activities to find what changed, user data to read chats and their files, and org data to attribute activity to the right people. Leave `delete:compliance_user_data` unselected, since Origin does not use it. Scopes are fixed once the key exists, so if one is missing, delete the key and create a new one.

---

## 

Step 3: Save the key in Origin

1. Sign in to the Origin dashboard at [dashboard.originhq.com](https://dashboard.originhq.com) as an admin.
2. Open **Settings > Integrations > Anthropic Compliance API key**.
3. Paste the key into the **Compliance Access Key** field.
4. Click **Save key**.

Origin stores the key encrypted and does not show it again. Once it is saved, the section reads **Configured** and reports the sync status.

---

## 

What to expect

Origin begins pulling within a few minutes. New Claude activity then flows continuously into your traces, analytics, and inventory, alongside the endpoint record. No further setup is required.

---

## 

Changing or removing the connection

- Rotate the key: create a new Compliance Access Key in claude.ai, then click **Replace key** in Origin.
- Disconnect: click **Remove** in Origin. New activity stops syncing; data already in Origin is unaffected.
- Revoke access: delete the key in claude.ai under **Settings > Data and Privacy > Compliance access keys**.

---

## 

Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Still reads "Not configured" after saving | The field was empty or the save did not complete | Paste the key again and click **Save key**. |
| Sync reports a 403 | The Compliance API is not enabled, or the key is missing a scope | Confirm the Compliance API is enabled in claude.ai and the key holds all three read scopes. If a scope is missing, create a new key. |
| No Claude activity appears | The key is valid but there is no recent Claude usage, or none within the backfill window | Use Claude, then wait for the next sync. |
| Sync stops after working | The key was deleted or revoked in claude.ai | Create a new key and click **Replace key** in Origin. |

---

## 

What Origin accesses

- Read access to your Claude organization through the three scopes above: chats, messages, files, projects, artifacts, and activity feed events.
- No delete access, and no ability to change anything in your Claude organization.
- The Compliance Access Key is stored encrypted and is never shown again after you save it.

Copy Page