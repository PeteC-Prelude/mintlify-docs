# Source: https://support.originhq.com/docs/entra-id-setup

This guide walks you through registering an application in your Microsoft Entra ID (formerly Azure AD) tenant, granting the permissions Origin needs, and connecting the integration from the Origin dashboard. After the first sync your users, groups, group memberships, devices (both Entra-joined and Intune-managed), and device-to-user assignments are available in Origin's Directory Browser.

The whole flow takes about ten minutes and requires a Microsoft work account with the **Global Administrator**, **Privileged Role Administrator**, or **Cloud Application Administrator** role — anyone who can grant tenant-wide admin consent for an application.

---

## 

What you'll end up with

| Synced from Microsoft Entra | Origin surface |
| --- | --- |
| Users (`/v1.0/users`) | Directory Browser → Users tab |
| Groups (`/v1.0/groups`) | Directory Browser → Groups tab |
| Direct user → group memberships | Click a user → "Groups" panel |
| Direct group → group nesting | Click a group → "Sub-groups" panel |
| Entra-joined / registered devices (`/v1.0/devices`) | Directory Browser → Devices tab |
| Intune-managed devices (`/v1.0/deviceManagement/managedDevices`) | Directory Browser → Devices tab (with serial numbers) |
| Endpoint ↔ user correlation by serial number or hostname | Endpoint detail panel "Assigned identity" |

Initial sync runs as soon as you finish step 7. After that, Origin re-syncs every hour, plus an on-demand button in the dashboard.

---

## 

Before you begin

You need:

1. **An Entra admin account** with one of the roles listed above.
2. **The Origin dashboard URL** for your tenant — the public hostname you sign in to. The redirect URI you'll register in Entra is `https://<your-origin-hostname>/integrations/oauth-callback`. If you sign in at `https://app.origin.example.com`, the redirect URI is `https://app.origin.example.com/integrations/oauth-callback`.
3. **Your Origin tenant admin role** — only Origin admins can create directory integrations.

You do **not** need:

- A pre-configured service principal. The setup below creates one.

What's degraded on the free tier:

- **Last sign-in time** (`last_login` in the directory browser) is sourced from Entra's `signInActivity` resource, which requires an **Entra ID P1 or P2** license plus the `AuditLog.Read.All` Graph permission. On the free tier — or on P1/P2 without `AuditLog.Read.All` granted — Graph silently omits the field and Origin records the user's last sign-in as the Unix epoch (`1970-01-01`). Stale-account detection that relies on this field will be best-effort until both prerequisites are in place. Everything else (users, groups, memberships, devices, owners) is available on the free tier with the permissions in step 2 below.

---

## 

Step 1 — Register the application in Entra

Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com) with your admin account, then:

1. **Identity** → **Applications** → **App registrations** → **New registration**.
2. Fill in the form:
 - **Name** — something a future admin will recognize, e.g. `Origin Directory Sync`.
 - **Supported account types** — _Accounts in this organizational directory only (Single tenant)_.
 - **Redirect URI** — select **Web** and enter `https://dashboard.originhq.com/integrations/oauth-callback`. The hostname must match exactly (scheme, host, port, path). Origin rejects callbacks whose redirect URI doesn't match what was stamped into the OAuth state.
3. Click **Register**.

You'll land on the new app's **Overview** page. Two values from this page go into the Origin dashboard later:

- **Application (client) ID**
- **Directory (tenant) ID**

Keep the tab open.

---

## 

Step 2 — Grant Microsoft Graph API permissions

Origin uses **application** permissions (also called _app-only_ or _client\_credentials_), not delegated permissions, so the sync continues to run when no user is signed in.

1. In the same app registration, click **API permissions** → **Add a permission** → **Microsoft Graph** → **Application permissions**.
2. Add the six permissions in the table below. You can search by name; tick the box and click **Add permissions** between groups if it's easier.

| Permission | Why Origin needs it |
| --- | --- |
| `User.Read.All` | Enumerate users for the directory browser and the user → groups join. |
| `Group.Read.All` | Enumerate groups, group members, and nested-group edges. |
| `Directory.Read.All` | Stable cross-resource directory reads (covers `/v1.0/organization` and the delta endpoints). |
| `Device.Read.All` | Enumerate Entra-joined and Entra-registered devices via `/v1.0/devices`. |
| `DeviceManagementManagedDevices.Read.All` | Enumerate Intune-managed devices via `/v1.0/deviceManagement/managedDevices` — this is what brings hardware serial numbers in for endpoint correlation. |
| `User.Read` (delegated, added by default) | Sign-in completion when an admin grants consent. Already present after registration; leave it in place. |

3. After all six are listed, click **Grant admin consent for `<your tenant>`** at the top of the table. You'll be prompted to confirm — accept. Each row should now show a green check under **Status**.

> 📘
> 
> ### 
> 
> **Tip.** If you see "Not granted for `<tenant>`" with a yellow warning after clicking Grant, your account doesn't have one of the admin roles listed in the prerequisites. Either get the role assignment or hand the rest of this guide to someone who has it; only the consent step is gated on it.

---

## 

Step 3 — Confirm the redirect URI

Step 1 already added one, but it's worth verifying — a typo here is the single most common reason the connect-to-Origin step fails.

1. **Authentication** → under **Web** → **Redirect URIs**.
2. Confirm `https://dashboard.originhq.com/integrations/oauth-callback` is listed exactly as you typed it.
3. Leave everything else on this page at its default. Origin does not need the implicit grant flow, ID tokens, or any extra "Front-channel logout" URI.
4. Click **Save** if you changed anything.

---

## 

Step 4 — Create a client secret

The secret is what the Origin backend presents to Microsoft to acquire Graph tokens after admin consent completes. It's stored on Origin's side encrypted under your tenant's KMS-managed data key — Origin admins never see it in plaintext after entry.

1. **Certificates & secrets** → **Client secrets** → **New client secret**.
2. **Description** — something descriptive, e.g. `origin-directory-sync-2026-q2`.
3. **Expires** — pick the longest window your security policy allows. Origin will warn you in the dashboard 14 days before the secret expires (the integration's status flips from **Connected** to **At risk**) so you can rotate before sync stops.
4. Click **Add**.

The secret value is shown **once** under the **Value** column. Copy it to your clipboard now — once you navigate away the column shows only the last few characters and you'll need to create a new secret to recover. The **Secret ID** column is not the secret; you don't need it.

---

## 

Step 5 — Connect from the Origin dashboard

1. Sign in to your Origin dashboard at `https://dashboard.originhq.com/`.
2. **Directory Integration** in the left sidebar → **\+ Add Integration** (or the empty-state **Add your first integration** button if this is the first one).
3. Pick **Entra ID** (Microsoft Entra ID).
4. Fill in the form:
 - **Display Name** — anything; this is just a label in the integrations list.
 - **Tenant ID** — paste the Directory (tenant) ID from step 1.
 - **Client ID** — paste the Application (client) ID from step 1.
 - **Client Secret** — paste the **Value** you copied in step 4.
5. Click **Connect with Entra ID**.

You'll be redirected to Microsoft to grant admin consent. Pick the admin account you used in steps 1–4 (or sign in if prompted), review the permission list — it should match the six you added in step 2 — and click **Accept**.

When you land back at the Origin dashboard, the integration row shows status **Connected**. If it shows **No credentials stored for this integration** instead, the secret didn't make it through; delete the row, return to step 4 to create a fresh secret, and re-run step 5.

---

## 

Step 6 — Trigger your first sync

Origin runs a periodic background sync, but you don't need to wait — click **Sync** in the row's **Actions** column. The status flips to **Syncing**, then back to **Connected** with a populated **Last Sync** column. A typical first sync takes a few seconds for small tenants and a couple of minutes for tenants with tens of thousands of users.

If the sync turns the row red and shows an error, see _Troubleshooting_ below.

---

## 

Step 7 — Verify in the Directory Browser

1. **Directory Integration** → **Directory Browser** tab.
2. The **Users** tab should list everyone Origin pulled.
3. Click any user. The detail panel that opens shows their direct groups under a **Groups** subsection — click any group to deep-link into that group's detail.
4. **Groups** tab → click any group. The detail panel shows direct user members and direct sub-groups (if your tenant uses nested groups). Click any member to jump back to that user.
5. **Devices** tab — Intune-managed devices show their serial number; Entra-joined devices that Intune hasn't seen show a blank serial and the device's `displayName`. Both share an **Owner** column resolved against the synced users.

If you've registered endpoints in Origin and the device serial numbers match, you'll see those endpoints' rows in the dashboard light up with the matched directory user under an **Assigned identity** column.

---

## 

What gets synced (and what doesn't)

**Synced — full list:**

- Users (active and disabled). Display name, email, UPN, department, job title, office, manager, last sign-in (best-effort — see _Before you begin_), account-enabled flag.
- Groups. Display name, description, type (security / distribution / Microsoft 365), member count, direct memberships, direct sub-groups.
- Entra-joined devices. Display name, OS, OS version, registered owners.
- Intune-managed devices. Hardware serial number, model, manufacturer, assigned UPN, compliance state, OS, OS version.

**Not synced:**

- Photos, contacts, mailbox metadata, OneDrive, calendars, or any tenant data outside the directory itself.
- Conditional Access policies, role assignments, or audit logs.
- Service principals or managed identities.
- Personal-access guest accounts (`#EXT#` UPN suffix). They're filtered out at sync time so they don't appear as "users" in the directory browser.

---

## 

Troubleshooting

### 

Connect-to-Azure-AD fails immediately with a 400

Either the tenant ID, client ID, or redirect URI is wrong. Double-check that:

- The redirect URI in Entra exactly matches `https://dashboard.originhq.com/integrations/oauth-callback`.
- The tenant ID and client ID in the dashboard form match the **Directory (tenant) ID** and **Application (client) ID** on the app's **Overview** page (not the Object ID — that's a different value).

### 

Admin consent succeeds but Origin shows "No credentials stored"

The client secret didn't reach Origin's backend. Most likely you left the **Client Secret** field blank, or pasted the **Secret ID** (the GUID in the right-hand column on Entra's secrets page) instead of the **Value**. Delete the integration in Origin, generate a fresh client secret in Entra, and run step 5 again.

### 

Sync row is red with "Azure token acquisition failed"

The stored secret is rejected by Microsoft's token endpoint. Common causes:

- The secret expired. Check **Certificates & secrets** in Entra; if the **Expires** column is in the past, generate a new one (step 4) and use the integration's **Edit** action to update the secret.
- The secret was revoked. Same fix.
- The application was deleted from Entra. Re-register from step 1.

### 

Sync runs but users / groups stay at zero

If only devices land and users/groups are empty, your app registration is probably missing the `User.Read.All` and `Group.Read.All` **application** permissions, or admin consent wasn't granted for them. Return to step 2, confirm both rows have green checks under **Status**, and run the sync again.

### 

Sub-groups column is always empty

Your tenant may not actually use nested groups — many small tenants don't. To confirm, in Entra navigate to one of your groups → **Members** and look for entries with the **Type** column set to **Group**; if there are none, there are no sub-groups for Origin to surface.

---

## 

Rotating the client secret

Origin shows a yellow **At risk** badge on the integration when the secret is within 14 days of expiry. To rotate without dropping sync:

1. Generate a new client secret in Entra (step 4). Don't delete the old one yet.
2. In the Origin dashboard, click **Edit** on the integration row, paste the new **Value** into the **Client Secret** field, and save.
3. Trigger a manual sync (step 6). If it succeeds, the new secret is live.
4. Delete the old secret from Entra.

---

## 

Removing the integration

To stop syncing without revoking access:

1. **Directory Integration** → click **Delete** in the row's **Actions** column. Origin tears down its end and tombstones the directory data (it's eligible for hard-delete after the retention window in your tenant settings, default 90 days).
2. Optionally also remove admin consent from your tenant: in Entra, **Identity** → **Applications** → **Enterprise applications** → pick your **Origin Directory Sync** app → **Permissions** → **Review permissions** → **This application has more permissions than I want it to have**. That revokes the granted permissions but leaves the app registration in place so you can re-grant later.
3. To fully remove the app, delete it from **App registrations** in Entra.

Copy Page