# Source: https://support.originhq.com/docs/macos-via-intune

A step-by-step walkthrough for silent deployment of the Origin macOS agent — configuration profiles, JWT staging script, and end-to-end Microsoft Intune workflow.

## 

Step 01 — Prerequisites

Confirm the following before proceeding:

- **Microsoft Intune admin center access** — Intune Administrator or equivalent role with permission to create configuration profiles, upload apps, and assign deployments.
- **An Apple MDM Push Certificate** that is active in Intune (**Devices → Enrollment → Apple → Apple MDM push certificate**). Without this, no macOS device can be managed.
- **A valid Origin provisioning JWT** (install token) — obtain from the Origin console under **Settings → Installers**.
- **The Origin macOS installer** — download from the Origin console under **Settings → Installers**. Delivered as `Origin.pkg`.
- **The Origin proxy root CA certificate** — generated in the Origin console under **Settings → Proxy CA Certificate**.
- **The four configuration profiles and install script** bundled with this guide (see files inventory in Step 03).
- **Target macOS devices enrolled in Intune with User-Approved MDM (UAMDM)**. Devices enrolled via Company Portal or Apple Automated Device Enrollment satisfy this requirement.
- **An Entra ID security group** containing your target devices — used during assignment in Step 05.
- **A pilot device** dedicated to validating the deployment chain end-to-end before broader rollout. The first device that receives a PKG app deployment in any tenant is at higher risk of agent-startup timing issues — see the IME warmup substep in Step 05.

> 📘
> 
> ### 
> 
> **Where to find it**
> 
> Both the `.pkg` installer and the provisioning JWT live in the same place in the Origin console: **Settings → Installers**. The tenant root CA is generated under **Settings → Proxy CA Certificate** — download the `.der` version. Grab all three before moving into Intune.
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
| `origin-vpn.mobileconfig` | VPN / transparent proxy configuration. Use as-is — no per-tenant customization required. |
| `origin-privacy.mobileconfig` | PPPC profile — grants Full Disk Access and Endpoint Security to the Origin agent. |
| `intune-warmup.sh` | One-time smoke test that confirms the Intune Management Agent (IME) is installed and reporting cleanly. Deployed via **Devices → Scripts and remediations** before Origin assignments to surface IME health issues early. |
| `intune-stage-jwt-origin.sh` | Standalone shell script that stages the provisioning JWT to `/var/tmp/origin-provisioning-jwt`. Deployed via **Devices → Scripts and remediations** before the macOS app (PKG) deployment. Idempotent — safe to retry. |

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
5. **JWT staging shell script** (`intune-stage-jwt-origin.sh`) — Stages the provisioning JWT to `/var/tmp/origin-provisioning-jwt` via a standalone shell script policy at **Devices → Scripts and remediations**. Must show **Succeeded** on a target device before that device is added to the macOS app (PKG) assignment. Idempotent — safe to re-run.
6. **The Origin .pkg (via macOS app deployment)** (`Origin.pkg`) — The installer itself. Deployed as a macOS app (PKG) with no pre-install script. The `.pkg` postinstall reads the JWT staged in step 5 and registers silently. See Step 05 for the full app setup.

## 

Step 05 — Intune Walkthrough

The following sequence assumes your tenant CA has already been generated in the Origin console and you have all four `.mobileconfig` files plus the `.pkg` and the two shell scripts (warmup and JWT staging) staged locally. The whole flow happens inside the Microsoft Intune admin center at `intune.microsoft.com`.

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

All four `.mobileconfig` files must be uploaded as **Custom** configuration profiles. Do not use Intune's built-in System Extensions, PPPC, or VPN templates — they reconstruct the XML and break the code requirements that make PPPC and system-extension allow-listing work.

> 📘
> 
> ### 
> 
> **Where to upload**
> 
> Intune admin center → **Devices → By platform → macOS → Configuration → Create → New Policy**. Set Profile type to **Templates**, then choose **Custom**. This is the only profile type that ingests a raw `.mobileconfig` without rewriting it.

1. Intune admin center → **Devices → By platform → macOS → Configuration**.
2. Click **\+ Create → New Policy**.
3. Set Profile type to **Templates**, select **Custom**, click **Create**.
4. **Basics**: Name the profile (e.g. `Origin — Proxy Root CA`) and click **Next**.
5. **Configuration settings**:
 - Custom configuration profile name: a friendly name shown on the device (e.g. `Origin Proxy Root CA`).
 - Deployment channel: **Device channel** — required for all four Origin profiles. The user channel will not allow the system extension or PPPC payloads to take effect.
 - Configuration profile file: upload `origin-proxy-ca.mobileconfig`.
6. Click **Next**.
7. **Assignments**: add your Entra ID device group under **Included groups**. Click **Next**.
8. **Review + create → Create**.
9. Repeat steps 2–8 for `origin-system-extensions.mobileconfig`, `origin-vpn.mobileconfig`, and `origin-privacy.mobileconfig` — in that order. Suggested names:
 - `Origin — System Extensions`
 - `Origin — Transparent Proxy`
 - `Origin — Privacy Preferences (PPPC)`

> ❗️
> 
> ### 
> 
> **Verify before proceeding**
> 
> Before moving to the package, confirm all four Origin profiles are installed using either method:
> 
> **Admin center:** Devices → All devices → select your test Mac → **Device configuration**. All four profiles should show status **Succeeded**.
> 
> **On the Mac:** System Settings → General → Device Management, or run:
> 
> Bash
> 
> ```
> sudo profiles show -all | grep -i "origin"
> ```
> 
> This should return entries for all four profiles. Do not proceed until all four are confirmed installed on the target device.

### 

02\. Verify Intune Management Agent reporting (warmup)

Before deploying the JWT staging script or the Origin `.pkg`, deploy a benign warmup script to confirm the Intune Management Agent (IME) is installed on your target devices and that result-reporting is healthy. The first PKG- or script-type assignment to a fresh tenant or device triggers the IME install, and the agent-startup window is the source of most "phantom failure" reports. Burning that timing budget on a no-op script is much cheaper than burning it on the JWT staging script and ending up with a cached failure to clean up.

> 📘
> 
> ### 
> 
> **Why this step exists**
> 
> The IME on macOS has two halves: a system-context daemon (`IntuneMdmDaemon`) and a user-context agent (`IntuneMdmAgent`). Scripts execute via the daemon, but result-reporting back to the Intune service goes through the agent. On a fresh enrollment the daemon comes up first; the agent only starts after a user signs into Company Portal. If a real assignment fires during that gap, the script can succeed on disk while reporting as failed in the admin center — and the cached failure is sticky. The warmup script's only job is to absorb that timing window with a no-op so subsequent real deployments report cleanly.

1. Intune admin center → **Devices → Scripts and remediations → Platform scripts tab → + Add → macOS**.
2. **Basics:** Name `Origin — IME Warmup`. Click **Next**.
3. **Script settings:** upload or paste the warmup script (below). Settings:
 - Run script as signed-in user: **No** (run as root).
 - Hide script notifications on devices: **Yes**.
 - Script frequency: **Not configured** (run once).
 - Max number of times to retry if script fails: **3**.

Bash

```
#!/bin/bash
# Origin Agent -- Intune Management Agent warmup smoke test.
# Confirms the IME is installed and result-reporting works correctly
# before any real Origin deployment runs.
LOG="/Library/Logs/Origin/intune-warmup.log"
mkdir -p "$(dirname "$LOG")"
exec >>"$LOG" 2>&1
echo "--- $(date -u +%Y-%m-%dT%H:%M:%SZ) warmup running ---"
echo "Hostname: $(hostname). User context: $(whoami)."
echo "Warmup complete."
exit 0
```

4. **Assignments:** add your Entra ID device group. Click **Next**.
5. **Review + add → Add**.
6. Force a sync from Company Portal on your pilot device. Wait for the script's status to show **Succeeded** in **Devices → Scripts and remediations → Origin — IME Warmup → Device status**.
7. Do not proceed to Substep 03 until status shows **Succeeded**. A **Failed** warmup means the IME isn't healthy on this device — diagnose before continuing (see Step 07).

### 

03\. Deploy the JWT staging shell script

This script stages the provisioning JWT to `/var/tmp/origin-provisioning-jwt` so the Origin `.pkg` postinstall can read it silently when the package installs. It is deployed as its own shell script policy — not as a pre-install script attached to the macOS PKG app.

1. Intune admin center → **Devices → Scripts and remediations → Platform scripts tab → + Add → macOS**.
2. **Basics:** Name `Origin — Stage Provisioning JWT`. Click **Next**.
3. **Script settings:** paste the script below, replacing the empty `JWT=""` value with your provisioning token from **Settings → Installers** in the Origin console. Settings:
 - Run script as signed-in user: **No** (run as root).
 - Hide script notifications on devices: **Yes**.
 - Script frequency: **Not configured** (run once).
 - Max number of times to retry if script fails: **3**.
4. **Assignments:** add your Entra ID device group. Click **Next**.
5. **Review + add → Add**.
6. Force a sync from Company Portal on your pilot device. Wait for status to show **Succeeded**.

Bash

```
#!/bin/bash
# Origin Agent -- Intune JWT Staging Script
# Stages the provisioning JWT to /var/tmp/origin-provisioning-jwt.
# Deploy via: Devices > Scripts and remediations > macOS
LOG="/Library/Logs/Origin/intune-stage-jwt.log"
JWT_FILE="/var/tmp/origin-provisioning-jwt"
####################################################################
# EDIT THIS: paste your provisioning JWT between the quotes
JWT=""
####################################################################
# All output goes to the log file -- never to stdout.
mkdir -p "$(dirname "$LOG")"
exec >>"$LOG" 2>&1
echo "--- $(date -u +%Y-%m-%dT%H:%M:%SZ) starting ---"
# Idempotency: if Origin is already running, no-op success.
if pgrep -x origin >/dev/null 2>&1; then
  echo "Origin agent already running; skipping JWT staging."
  exit 0
fi
# Idempotency: if JWT already staged with content, no-op success.
if [[ -s "$JWT_FILE" ]]; then
  echo "JWT already staged; skipping."
  exit 0
fi
# Validate JWT is configured.
if [[ -z "$JWT" ]]; then
  echo "ERROR: JWT not configured. Edit the script body."
  exit 1
fi
# Stage the JWT atomically.
umask 077
printf '%s' "$JWT" > "$JWT_FILE.tmp"
chmod 600 "$JWT_FILE.tmp"
mv -f "$JWT_FILE.tmp" "$JWT_FILE"
echo "JWT staged successfully ($(wc -c < "$JWT_FILE") bytes)."
exit 0
```

On the device, verify the JWT was actually staged: `sudo ls -la /var/tmp/origin-provisioning-jwt` should show a `0600`\-mode file owned by `root:wheel` with content. Until this is confirmed, do not proceed to Substep 04.

> 📘
> 
> ### 
> 
> **Why the token lives in the script body**
> 
> Intune doesn't expose script parameters for macOS shell scripts the way it does for Windows Win32 apps — the script body is the only place to put the JWT. Any Intune admin with read permission on this script policy can see it. Scope admin permissions accordingly. For organizations where this exposure is unacceptable, see the tenant-signed PKG path that eliminates JWT staging entirely.

### 

04\. Add the Origin package as a macOS PKG app

The Origin `.pkg` is deployed using Intune's **macOS app (PKG)** deployment type. The PKG app has **no pre-install or post-install script** — JWT staging was handled in Substep 03. The `.pkg` postinstall reads the staged JWT directly when the package installs.

> 📘
> 
> ### 
> 
> **Intune Management Agent prerequisite**
> 
> The macOS app (PKG) deployment type requires the Microsoft Intune Management Agent (version 2309.007 or greater) on the endpoint. The IME should already be present from Substep 02 (warmup script). If it isn't, that's a sign Substep 02 didn't actually succeed and you should diagnose before continuing.

### 

05\. Configure requirements, detection, and assignment

1. Intune admin center → **Apps → All apps → + Create**.
2. Under **Other**, select **macOS app (PKG)**, then click **Select**.
3. **App information → Select app package file:** upload `Origin.pkg`. Intune reads bundle ID and version from the package metadata.
4. Confirm the auto-populated values, fill in **Publisher** (e.g. Prelude Research) and any optional fields. Click **Next**.
5. **Program:** leave both **Pre-install script** and **Post-install script** blank. This is critical — adding a pre-install script reintroduces the failure modes documented in Step 07.
6. Set **Ignore app version** to **No** so Intune respects the version in the package metadata for upgrade decisions.
7. Click **Next** to continue to **Requirements**.

| Setting | Value |
| --- | --- |
| Minimum operating system | macOS 13.0 or later (matches Origin's supported OS floor) |

**Detection rules**

| Field | Value |
| --- | --- |
| Rules format | Use included app rules (default) — Intune detects via the bundle IDs included in the `.pkg` |
| Action | Leave defaults — Intune populates from the package |

**Assignments**

| Field | Value |
| --- | --- |
| Required | Add the same Entra ID device group used for the configuration profiles |
| Available for enrolled devices | Leave empty (this would let users self-install via Company Portal — not what we want for a silent rollout) |
| Uninstall | Not supported for macOS PKG apps deployed via Intune Management Agent |

Review the values and click **Create**.

> ❗️
> 
> ### 
> 
> **Required ordering before assigning the PKG app**
> 
> Intune has no built-in dependency mechanism between scripts and apps. Order is enforced by gating which devices receive each assignment:
> 
> 1. The four configuration profiles and the warmup script (Substep 02) should be assigned to your full pilot/rollout group from the start.
> 2. The JWT staging script (Substep 03) should also be assigned broadly — it's idempotent, so over-assigning is safe.
> 3. The macOS PKG app (this substep) should be assigned to a narrower sub-group that you populate only after a device shows **Succeeded** on all of: the four profiles, the warmup script, and the JWT staging script. For pilot validation, populate the sub-group manually one device at a time. For broader rollout, automate population via a dynamic group whose membership rule keys off a custom attribute the JWT script writes on success.

### 

06\. Trigger the install

Once a device is in scope for the macOS PKG app (with all upstream profiles and scripts already **Succeeded**), the install runs automatically on the next device check-in. To force it for testing:

- **From the Mac:** open Company Portal → Devices → select the device → ⋯ → **Check status**. This forces an immediate sync.
- **From the admin center:** Devices → All devices → select the target device → **Sync**.
- **From Terminal on the Mac** (forces a profile check-in only — does not directly trigger app install): `sudo profiles renew -type enrollment`

**End-to-end flow:** configuration profiles apply → warmup confirms IME health → JWT staging script writes `/var/tmp/origin-provisioning-jwt` → Intune Management Agent installs the `.pkg` → the postinstall reads the staged JWT, registers silently, and deletes the file. No user interaction is required at any step.

## 

Step 06 — Verification

Run these on a target endpoint after the install completes. Each command confirms a different stage of the install pipeline is healthy.

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

**Review agent log**

Bash

```
cat /Library/Origin/logs/agent.log.$(date -u +%Y-%m-%d)
```

Prints today's agent log. Useful for confirming the agent has started cleanly, registration succeeded, and there are no errors after install.

**JWT staging script log**

Bash

```
sudo cat /Library/Logs/Origin/intune-stage-jwt.log
```

Should show a successful run with `JWT staged successfully (NNN bytes).` or one of the idempotency no-op messages if the JWT was already staged or Origin was already running.

**Intune Management Agent log**

Bash

```
sudo tail -n 100 /Library/Logs/Microsoft/Intune/IntuneMDMDaemon*.log
```

Confirms the package installed cleanly. Look for `App install succeeded` for the Origin policy ID. If you see `Cached app policy result matches` repeatedly without resolution, see Step 07.

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

> 📘
> 
> ### 
> 
> **Intune admin center verification**
> 
> Devices → All devices → select the target Mac → **Managed Apps**. The Origin app should show install status **Installed**. Under **Device configuration**, all four Origin profiles should show **Succeeded**.

## 

Step 07 — Troubleshooting

### 

Profile shows "Error" or "Conflict" in Intune

Check the per-device status in **Devices → All devices → \[device\] → Device configuration → \[profile name\] → Per-setting status**. The two most common causes:

- **Profile uploaded with the wrong template type** — if you used Intune's built-in PPPC or System Extensions template instead of Templates → Custom, the XML was rebuilt and the code requirement no longer matches Origin's signature. Delete the profile and re-create it as a Custom profile uploading the raw `.mobileconfig`.
- **User channel selected instead of Device channel** — system extension and PPPC payloads are device-scoped and silently no-op when delivered through the user channel. Recreate the profile with Deployment channel: **Device channel**.
- **Profile stuck in "Pending"** — force a sync from Company Portal on the device, or use **Sync** in the admin center.
- Verify the profile actually installed: `sudo profiles show -all | grep -i "origin"`.

### 

Full Disk Access not granted despite profile

- The PPPC profile's code requirement matches Developer ID signed builds with Team ID `D3C73MWD7Y`. Ad-hoc signed or internal development builds won't match and FDA will not pre-grant.
- Confirm the profile was uploaded as a Custom template (see above) — the built-in PPPC template will silently strip the code requirement.

### 

JWT staging script reports "Failed" but the JWT file exists

Diagnostic and fix for the case where the script ran but Intune reports failure:

- Verify the file actually exists with content: `sudo ls -la /var/tmp/origin-provisioning-jwt`. A `0600`\-mode file owned by `root:wheel` with non-zero size means the script succeeded — Intune is misreporting.
- Check the script's own log: `sudo cat /Library/Logs/Origin/intune-stage-jwt.log`. If the log shows `JWT staged successfully` and `exit 0`, the script worked.
- **Recovery:** if the JWT is staged correctly, simply assign the macOS PKG app to the device — the postinstall will consume the JWT regardless of what Intune thinks of the script's status.
- If you need Intune to actually report success, toggle the script's group assignment off, wait 5 minutes, then re-assign. This forces Intune to re-evaluate from a fresh state. The script's idempotency check ("JWT already staged; skipping") returns 0 cleanly.

### 

PKG app reports "Pre-install script did not complete successfully (0X87D3014A)"

This means the macOS PKG app was configured with a pre-install script — which this deployment guide explicitly avoids. Open the app in **Apps → All apps → Origin app → Properties → Edit** and confirm both **Pre-install script** and **Post-install script** fields are blank. If they aren't, blank them, save, and re-sync.

### 

Cached failure persists across multiple sync attempts

Intune's macOS app retry logic has a documented weakness: when the `IntuneMDMDaemon`'s cached policy result matches a prior failure (visible as `Cached app policy result matches the current policy result` in `IntuneMDMDaemon*.log`), it stops retrying meaningfully. Recovery options in order of preference:

- **Toggle the assignment.** Apps → All apps → Origin → Properties → Assignments → remove the group → Save. Wait 5 minutes, force a sync from Company Portal, then re-add the assignment. Intune treats this as a brand-new policy and clears the cached result.
- **Retire and re-enroll.** If the assignment toggle doesn't break the cache, retire the device from the admin center, manually clean `/Library/Intune/` and `/Library/Logs/Microsoft/Intune/`, delete the device record from Intune, then re-enroll fresh via Company Portal. This is the only fully-clean reset for a poisoned policy state.
- **Wait it out (not recommended).** Intune may eventually break the cache on its own, but timing is unpredictable.

### 

Warmup script fails on first deployment

This indicates the Intune Management Agent itself isn't healthy on this device. Check `ls /Library/Intune/` — if empty or missing the agent app bundle, force a sync and wait 5–10 minutes for the agent to install.

### 

App stays in "Pending" forever in Company Portal

- Confirm the device's primary user has an Intune license assigned in M365 Admin Center. Without the license, agent installation requests get silently dropped.
- Check that the device hasn't been added to a conflicting assignment that excludes it from the script policy.
- This is a known Intune cosmetic bug — Company Portal can show **Pending** even after the app has successfully installed. Verify with `pgrep -x origin` on the endpoint and the install status in the admin center, both of which are authoritative.
- If the install really hasn't started, confirm the device received the Intune Management Agent: `ls /Library/Intune/`. If empty, force a sync and wait — the agent installs on first PKG app or shell script assignment.

Copy Page