# Source: https://support.originhq.com/docs/macos-via-jamf

A step-by-step walkthrough for silent deployment of the Origin macOS agent — configuration profiles, install script, and end-to-end Jamf Pro workflow.

## 

Step 01 — Prerequisites

Confirm the following before proceeding:

- **Jamf Pro admin access** (scope permitting profile creation, package upload, policies, and scripts).
- **A valid Origin provisioning JWT** (install token) — obtain from the Origin console under **Settings → Installers**.
- **The Origin macOS installer** — download from the Origin console under **Settings → Installers**. Delivered as `Origin.pkg`.
- **The Origin proxy root CA certificate** — generated in the Origin console under **Settings → Proxy CA Certificate**.
- **The four configuration profiles and install script** bundled with this guide (see files inventory in Step 03).
- **Target macOS devices enrolled in Jamf Pro** with User-Approved MDM (UAMDM).

> 📘
> 
> ### 
> 
> **Where to find it**
> 
> Both the `.pkg` installer and the provisioning JWT live in the same place in the Origin console: **Settings → Installers**. The tenant root CA is generated under **Settings → Proxy CA Certificate**. Grab all three before moving into Jamf.
> 
> Deployment package download — configuration profiles and install script: [drive.google.com/drive/folders/1r3PYJdVMgnTVjzyJqzKvys8rTFqWsKKF](https://drive.google.com/drive/folders/1r3PYJdVMgnTVjzyJqzKvys8rTFqWsKKF)

## 

Step 02 — Scope of Automation

Four macOS permissions stand between the installer and a fully registered agent. MDM pre-approves all four — no user interaction is required for any permission grant.

| Permission | Pre-approved by Profile? | User Action |
| --- | --- | --- |
| Full Disk Access | Yes | None |
| Endpoint Security Client | Yes | None |
| System Extension (DNS proxy) | Yes | None |
| Network proxy | Yes | None |

Deploying all four configuration profiles before the package installs ensures the agent can register silently without any user-facing prompts or permission dialogs.

## 

Step 03 — Files

All files are available for download from [drive.google.com/drive/folders/1r3PYJdVMgnTVjzyJqzKvys8rTFqWsKKF](https://drive.google.com/drive/folders/1r3PYJdVMgnTVjzyJqzKvys8rTFqWsKKF). The template profile requires per-tenant customization before upload — all others can be used as-is.

| File | Purpose |
| --- | --- |
| `origin-proxyca.mobileconfig.template` | Template for the tenant root CA profile. Requires per-tenant customization before upload. |
| `origin-systemextensions.mobileconfig` | System extension policy — pre-approves the DNS proxy and marks it non-removable by end users. |
| `origin-vpn.mobileconfig` | VPN / transparent proxy configuration. Compatible with Jamf Pro, Intune, Kandji, Mosyle, and other macOS MDMs. |
| `origin-privacy.mobileconfig` | PPPC profile — grants Full Disk Access and Endpoint Security to the Origin agent. |
| `jamf-install-origin.sh` | Pre-install shell script that stages the provisioning JWT for the `.pkg` postinstall. |

### 

Identifiers

| Identifier | Value |
| --- | --- |
| Team ID | `D3C73MWD7Y` |
| Agent bundle ID | `com.origin.agent` |
| DNS proxy bundle ID | `com.origin.dns-proxy` |

## 

Step 04 — Deployment Order

All configuration profiles must land on the endpoint before the `.pkg` installs — otherwise the agent hits permission prompts during registration and the silent-install chain breaks. Follow this order strictly:

1. **Proxy root CA certificate** (`origin-proxy-ca.mobileconfig`) — Installs the tenant-specific root CA into the macOS trust store. Without this, TLS inspection of AI provider traffic would trigger certificate warnings. Must be customized per tenant — instructions in Step 05.
2. **System extension policy** (`origin-system-extensions.mobileconfig`) — Pre-approves the Origin DNS proxy system extension (`com.origin.dns-proxy`) so it loads without a user prompt. Also marks it non-removable — end users can't disable it from System Settings.
3. **VPN / transparent proxy** (`origin-vpn.mobileconfig`) — Pre-creates the transparent proxy configuration that routes AI-destined traffic through the Origin DNS proxy.
4. **Privacy preferences (PPPC)** (`origin-privacy.mobileconfig`) — Grants Full Disk Access and Endpoint Security to the Origin agent based on Team ID `D3C73MWD7Y` and bundle ID `com.origin.agent`. Eliminates TCC prompts that would otherwise appear on first run.
5. **The Origin .pkg (via policy)** (`Origin.pkg`) — The installer itself. Deployed via a Jamf policy with the install script running before the package to stage the provisioning JWT. See Step 05 for the full policy setup.

## 

Step 05 — Jamf Pro Walkthrough

The following sequence assumes your tenant CA has already been generated in the Origin console and you have all four `.mobileconfig` files plus the `.pkg` and install script staged locally.

### 

00\. Prepare the proxy CA profile

The proxy CA profile is shipped as a template — embed your tenant's root CA before uploading.

1. In the Origin console, go to **Settings → Proxy CA Certificate**.
 
2. Click **Generate Root CA** if one hasn't been generated for your tenant yet.
 
3. Click **Download DER** to retrieve the root CA as `origin-proxy-root-ca.der`.
 
4. Base64-encode the DER file:
 
 Bash
 
 ```
    base64 < origin-proxy-root-ca.der
    ```
 
5. Open `origin-proxy-ca.mobileconfig.template` and replace `REPLACE_WITH_BASE64_ENCODED_DER_CERTIFICATE` with the output from step 4.
 
6. Generate a fresh UUID and replace both instances of `REPLACE_WITH_UNIQUE_UUID`:
 
 Bash
 
 ```
    uuidgen
    ```
 
7. Save the file as `origin-proxy-ca.mobileconfig`.
 

### 

01\. Upload configuration profiles

All four `.mobileconfig` files must be uploaded as **raw custom profiles**. Do not use Jamf's built-in PPPC or System Extensions payload editors — they rewrite the XML and break the code requirements.

> 📘
> 
> ### 
> 
> **Upload location**
> 
> The **Upload** button is on the Configuration Profiles list page (**Computers → Configuration Profiles**), not inside the New Profile editor. Navigate to the list first, then click **Upload** to import a raw `.mobileconfig` file.

1. Jamf Pro → **Computers → Configuration Profiles**.
2. Click **Upload** on the list page.
3. Upload `origin-proxy-ca.mobileconfig`.
4. Under the **Scope** tab, add your target devices or Smart Group.
5. Click **Save**.
6. Repeat for `origin-system-extensions.mobileconfig`, `origin-vpn.mobileconfig`, and `origin-privacy.mobileconfig` — in that order.

> ❗️
> 
> ### 
> 
> **Verify before proceeding**
> 
> Before moving to package installation, confirm all four Origin profiles are installed using either method:
> 
> **UI:** On the Mac go to **System Settings → Privacy & Security → Profiles** (or search "Device Management"). All four profiles — Proxy Root CA, System Extensions, VPN, and Privacy Preferences — should appear in the managed profiles list.
> 
> **Terminal:**
> 
> Bash
> 
> ```
> sudo profiles show -all | grep -i "origin"
> ```
> 
> This should return entries for all four profiles. Do not proceed until all four are confirmed.

### 

02\. Upload the Origin package

1. Jamf Pro → **Settings → Computer Management → Packages → Upload Package**.
2. Upload `Origin.pkg`.
3. Leave defaults (Category, Priority, Fill User Template, etc.) unless you have site-specific conventions.
4. Click **Save**.

### 

03\. Create the install script

The script stages the provisioning JWT to `/var/tmp/origin-provisioning-jwt` so the `.pkg` postinstall can read it silently. The file is deleted after use.

1. Jamf Pro → **Settings → Scripts → New**.
2. Name the script `Origin Installer`.
3. On the **Script** tab, paste the contents below, replacing the empty `JWT=""` value with your provisioning token from **Settings → Installers**.
4. Click **Save**.

Bash

```
#!/bin/bash
# Origin Agent -- Jamf Silent Install Script
# Stages the JWT for the .pkg postinstall to read.
# The postinstall deletes the file after use.
set -euo pipefail
####################################################################
# EDIT THIS: paste your provisioning JWT between the quotes
JWT=""
####################################################################
if [[ -z "$JWT" ]]; then
  echo "Error: JWT not configured. Edit the script and set the JWT variable."
  exit 1
fi
JWT_FILE="/var/tmp/origin-provisioning-jwt"
install -m 600 /dev/null "$JWT_FILE"
echo "$JWT" > "$JWT_FILE"
echo "JWT staged for Origin installer."
```

> 📘
> 
> ### 
> 
> **Why the token lives in the script body**
> 
> Jamf script parameters are capped at 255 characters, which is too short for a provisioning JWT. The token goes directly into the script body. Any Jamf admin with script-view permissions will be able to read it — scope admin permissions accordingly.

### 

04\. Create the install policy

1. Jamf Pro → **Computers → Policies → New**.

**General Tab**

| Field | Value |
| --- | --- |
| Display Name | Install Origin Agent |
| Trigger | Custom event: `install-origin` and **Enrollment Complete** |
| Execution Frequency | Once per computer |

**Packages Tab**

| Field | Value |
| --- | --- |
| Package | `Origin.pkg` |
| Action | Install |

**Scripts Tab**

| Field | Value |
| --- | --- |
| Script | Origin Installer |
| Priority | **Before** |

**Scope Tab**

| Field | Value |
| --- | --- |
| Target | Add the same target devices or Smart Group used for the configuration profiles. Click **Save**. |

> ❗️
> 
> ### 
> 
> **Script priority must be Before**
> 
> If the script runs **After** the package, the JWT file won't exist when the postinstall needs it and the installer will fall back to the GUI prompt. Double-check the priority before saving.

### 

05\. Trigger the policy

On a target device, run the custom trigger from Terminal:

Bash

```
sudo jamf policy -event install-origin
```

> 📘
> 
> ### 
> 
> **Authentication note**
> 
> Running `sudo jamf policy -event install-origin` manually will prompt for interactive password authentication on the endpoint. If you prefer the install to run without any user interaction, simply wait for the device to check in naturally (on enrollment complete or the next recurring check-in) rather than forcing it with the manual command.

The script stages the JWT → Jamf installs the `.pkg` → the postinstall reads the staged JWT and registers silently → the JWT file is deleted. The MDM profiles handle all permission grants with no user interaction required.

## 

Step 06 — Verification

Run these on a target endpoint after the policy completes. Each command confirms a different stage of the install pipeline is healthy.

**Profiles installed**

Bash

```
sudo profiles show -all | grep -i "origin"
```

Should return entries for all four Origin profiles: proxy CA, system extensions, VPN, and privacy preferences.

**System extension loaded**

Bash

```
systemextensionsctl list | grep com.origin.dns-proxy
```

Status should be `[activated enabled]`. Anything else indicates the system extension didn't approve cleanly.

**Agent process running**

Bash

```
pgrep -x origin
```

Returns a PID when the agent is running.

**Agent log tail**

Bash

```
cat /Library/Origin/logs/agent.log.$(date -u +%Y-%m-%d)
```

Prints today's agent log. Useful for confirming the agent has started cleanly, registration succeeded, and there are no errors after install.

### 

Origin menu bar icon

Once the agent is running, the Origin menu bar icon appears in the macOS menu bar. Click it to confirm the agent is active and to access two operator-friendly checks without dropping to Terminal:

- **Generate diagnostic report** — bundles agent state, recent logs, and registration status into a single file you can hand to Origin support.
- **View logs** — opens the local log directory directly, equivalent to the `cat` command above without needing the date.

> 📘
> 
> ### 
> 
> **Origin console verification**
> 
> In the Origin console, click **Computers** in the left-hand navigation, then search for the target hostname. A successfully registered endpoint will appear in the list with a recent snapshot time and its detected AI agents populated.

## 

Step 07 — Troubleshooting

### 

Profile shows "Failed" in Jamf

Check the error in the device's Management History. The two most common causes:

- **"AllowedTeamIdentifiers and AllowedSystemExtensions conflict"** — the profile was recreated in Jamf's payload editor. Re-upload the raw `.mobileconfig` from the deployment package instead.
- **Profile stuck pending** — run `sudo jamf policy` on the device to force a check-in.

### 

Full Disk Access not granted despite profile

- Verify the profile actually installed:
 
 Bash
 
 ```
    sudo profiles show -all | grep -i "origin"
    ```
 
- The PPPC profile's code requirement matches Developer ID signed builds with Team ID `D3C73MWD7Y`. Ad-hoc signed or internal development builds won't match and FDA will not pre-grant.
 

### 

JWT prompt still appears on install

- Verify the script ran successfully — check the policy logs in Jamf Pro.
- Confirm script priority is **Before**, not **After**.
- On the endpoint, confirm the file existed before the `.pkg` ran: `/var/tmp/origin-provisioning-jwt`.

## 

Step 08 — Alternative Deployment Paths

### 

Kandji

1. **Library → Add New → Custom Profile** for each `.mobileconfig`.
2. Assign to device blueprints.
3. Add the `.pkg` as a **Custom App**.
4. Use a pre-install script to stage the JWT (same pattern as the Jamf script above).

### 

Headless CLI install

For scripted deployments outside of any MDM — useful for lab environments, golden-image prep, or emergency reinstalls:

Bash

```
echo "eyJ..." > /var/tmp/origin-provisioning-jwt
chmod 600 /var/tmp/origin-provisioning-jwt
installer -pkg Origin.pkg -target /
```

The postinstall reads the JWT from the staged file and skips the GUI prompt. The file is deleted after use.

> 📘
> 
> ### 
> 
> **Note**
> 
> Without the configuration profiles, the end user will see all four permission prompts — this path is best reserved for managed endpoints that already have the profiles from an earlier MDM state, or lab machines where the prompts are acceptable.

Copy Page