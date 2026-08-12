# Source: https://support.originhq.com/docs/tanium-deployment-macos-agent

A step-by-step walkthrough for silent deployment of the Origin macOS agent — configuration profiles, install script, and end-to-end Tanium deployment flow.

## 

Overview

Deploying the Origin agent on macOS requires two Tanium products working in sequence:

- **Tanium Device Management** pushes the four MDM configuration profiles that pre-approve the system permissions the agent needs. Without these profiles the agent will hit interactive permission dialogs during registration and silent install will fail.
- **Tanium Deploy** stages the provisioning JWT and executes the installer, but only on endpoints where all four profiles have successfully installed.

The two products are independent — Device Management uses Apple's MDM/APNs channel, Deploy uses the Tanium Client. Both must be in place on the endpoint before the agent install runs.

---

## 

Prerequisites

Before starting, confirm the following:

- Tanium Cloud admin access with permissions to configure Device Management (`Shared Services → Device Management`) and Deploy (`Modules → Deploy`)
- A Tanium licence that includes **Tanium Enforce** (required by Device Management)
- An Apple Push Notification Service (APNs) certificate configured in Device Management — `Shared Services → Device Management → Settings → Push Certificate`
- Target Macs enrolled in Tanium Device Management (see Step 1)
- A valid Origin **provisioning JWT** — obtain from the Origin console under `Settings → Installers`
- The Origin **macOS installer** (`Origin.pkg`) — download from the Origin console under `Settings → Installers`
- The Origin **proxy root CA certificate** in DER format (`origin-proxy-root-ca.der`) — generated in the Origin console under `Settings → Proxy CA Certificate`
- The four `.mobileconfig` files from the deployment package download

**Deployment package download — configuration profiles:** 
`drive.google.com/drive/folders/1r3PYJdVMgnTVjzyJqzKvys8rTFqWsKKF`

---

## 

Files

| File | Purpose |
| --- | --- |
| `origin-proxyca.mobileconfig.template` | Template for the tenant root CA profile. Requires per-tenant customisation before upload — see Step 2. |
| `origin-systemextensions.mobileconfig` | Pre-approves the Origin DNS proxy system extension (`com.origin.dns-proxy`) and marks it non-removable. |
| `origin-vpn.mobileconfig` | VPN / transparent proxy configuration. Use this generic variant — **not** `origin-vpn-jamf.mobileconfig`. |
| `origin-privacy.mobileconfig` | PPPC profile — grants Full Disk Access and Endpoint Security to the Origin agent (Team ID `D3C73MWD7Y`, bundle ID `com.origin.agent`). |
| `Origin.pkg` | The macOS agent installer. |
| `jwt.txt` | A plain text file containing your provisioning JWT. Created by you — see Step 4. |

### 

Identifiers

| | |
| --- | --- |
| Team ID | `D3C73MWD7Y` |
| Agent bundle ID | `com.origin.agent` |
| DNS proxy bundle ID | `com.origin.dns-proxy` |
| App bundle | `/Applications/Origin.app` |
| CFBundleName | `Origin` |
| Current version | `1.1.0` |
| Minimum macOS | 11.0 (Big Sur) |

---

## 

Deployment Order

All four configuration profiles must reach **Installed** status on the endpoint before the Deploy package runs. The Deploy deployment is gated on this via a computer group (Step 3) — it will not run on a Mac until all four profiles are confirmed installed.

| Order | What | How |
| --- | --- | --- |
| 1 | Proxy Root CA profile | Device Management custom setting |
| 2 | System Extensions profile | Device Management custom setting |
| 3 | VPN / transparent proxy profile | Device Management custom setting |
| 4 | Privacy (PPPC) profile | Device Management custom setting |
| 5 | `Origin.pkg` via JWT staging | Tanium Deploy package |

---

## 

Step 1 — Enroll Macs in Device Management

Macs must be enrolled in Tanium Device Management before profiles can be pushed. Device Management enrollment also automatically installs the Tanium Client, which Deploy requires.

### 

Automated enrollment (recommended)

If your Macs are purchased through Apple Business Manager (ABM) or Apple School Manager (ASM), configure automated device enrollment so Macs enroll into Tanium Device Management at setup time with no user interaction required.

1. Go to `Shared Services → Device Management → Settings (GEAR icon not the side menu) → Automated Enrollment`
2. Follow the prompts to link your ABM/ASM token and configure the enrollment experience

### 

Manual enrollment (testing / pilot)

For individual Macs not in ABM, download and install the enrollment profile manually:

1. Go to `Shared Services → Device Management → Settings (GEAR icon not the side menu) → Device Enrollment`
2. In the **Manual Enrollment** section, click **Download** and select the macOS enrollment file
3. Copy the profile to the target Mac and install it
4. On the Mac, open the profile in System Settings and confirm installation

![](https://files.readme.io/ca82f9715480f3e9b0113541223825215605d914cf56e6f0a13e38df6324f408-image.png)

 
`[SCREENSHOT: Device Management → Settings → Device Enrollment showing the Manual Enrollment download button]`

Once enrolled, the Tanium Client installs automatically. Verify enrollment by checking that the Mac appears in `Shared Services → Device Management` with a recent check-in time.

Alternatively you can ask the question: 
`Get Computer Name and Device Management - Last Check-In Time and Device Management - Device ID from all entities with ( Is Mac equals true and Device Management - Device ID matches "^[\da-f]{8}-.*$" )`

![](https://files.readme.io/ecb93833389594c43660ac9e18792a7423e2cbd006a1de14da4db65ae854aae4-image.png)

 
`[SCREENSHOT: Question Response showing enrolled Mac]`

---

## 

Step 2 — Prepare and Upload the Configuration Profiles

All four profiles are uploaded as Custom Settings in Tanium Device Management. Device Management pushes them via the MDM/APNs channel — no Tanium Enforce payload editors are used.

**Do not recreate these profiles using Tanium Enforce's built-in payload editors.** The editors rewrite the XML and could break the code requirements in the PPPC and System Extensions profiles. Upload the raw `.mobileconfig` files only.

### 

00 — Prepare the proxy CA profile

The proxy CA profile ships as a template and requires per-tenant customisation before it can be uploaded.

**1\. Generate the root CA**

In the Origin console, go to `Settings → Proxy CA Certificate`. Click **Generate Root CA** if one has not been generated for your tenant, then click **Download DER** to save `origin-proxy-root-ca.der`.

**2\. Base64-encode the DER file**

Shell

```
base64 < origin-proxy-root-ca.der
```

Copy the entire output — this is your encoded certificate.

**3\. Edit the template**

Open `origin-proxyca.mobileconfig.template` and make the following replacements:

| Placeholder | Replace with |
| --- | --- |
| `REPLACE_WITH_BASE64_ENCODED_DER_CERTIFICATE` | The base64 output from step 2 |
| First `REPLACE_WITH_UNIQUE_UUID` | Output of `uuidgen` |
| Second `REPLACE_WITH_UNIQUE_UUID` | Output of a **second separate** `uuidgen` call |

The two UUIDs must be different from each other — one identifies the payload, the other identifies the profile envelope.

**4\. Make** `PayloadType` **first in** `PayloadContent` **array’s dictionary**

Tanium Device Management's validator appears to inconsistently require the `PayloadType` key to be first in the `PayloadContent`. Ensure the outer `PayloadContent`’s inner payload dict starts with `PayloadType`:

XML

```
<?xml version="1.0" encoding="UTF-8"?>
<plist version="1.0">
  <dict>
    <key>PayloadContent</key>
    <array>
      <dict>
        <key>PayloadType</key>
        <string>com.apple.security.root</string>
        <key>PayloadContent</key>
        <data>
```

Validate this if the upload fails with a 500 error and the message _"The profile is either missing some required information or contains information in an invalid format."_

**5\. Save as** `origin-proxy-ca.mobileconfig`

### 

01 — Upload all four profiles

Repeat the following for each profile, in order:

1. `origin-proxy-ca.mobileconfig`
2. `origin-systemextensions.mobileconfig`
3. `origin-vpn.mobileconfig`
4. `origin-privacy.mobileconfig`

For each:

1. Go to `Shared Services → Device Management → Settings`
2. In the **Custom Setting** section, click **+**
3. Name the setting — use the names in the table below
4. In **Choose a platform**, select **macOS**
5. In **Upload your profile**, drag and drop the `.mobileconfig` file
6. Click **Open**, then **Save**

![](https://files.readme.io/6637836b50ee04554e4c8291215dd1e1f5c7faee32d9123c0f3161971ec21703-image.png)

`[SCREENSHOT: Device Management → Settings (not GEAR) showing the Custom Settings upload dialog with a .mobileconfig file being dragged in]`

| File | Setting name |
| --- | --- |
| `origin-proxy-ca.mobileconfig` | `Origin - Proxy Root CA` |
| `origin-systemextensions.mobileconfig` | `Origin - System Extensions` |
| `origin-vpn.mobileconfig` | `Origin - VPN` |
| `origin-privacy.mobileconfig` | `Origin - Privacy` |

**Use these exact names.** The computer group created in Step 3 matches against these names in sensor output.

`![][image4]`

![](https://files.readme.io/40ba82e799c2a59783056a44f74a50ac8429fbceb4f00cb97bc74ecc62cb1ad5-image.png)

`[SCREENSHOT: Device Management → Settings showing all four Origin custom settings listed]`

---

## 

Step 3 — Create the Collection

A Device Management collection groups the four settings and targets them to the appropriate devices.

1. Go to `Shared Services → Device Management → Collections`
2. Click **Add Collection**
3. Name the collection `Origin Agent - Profiles`
4. Click **Add** for each of the four Origin settings and click **Next**
5. Skip the Applications step and click **Next**
6. Configure targeting — select the computer group containing your target Macs
7. Click **Preview Targeted Endpoints** to verify scope, then **Save**

![](https://files.readme.io/2e260fc2b5ea1791a8d74287b15aa3ee5879ed58a8071d03a076ccd4575a6326-image.png)

`[SCREENSHOT: Device Management → Collections showing the Origin Agent - Profiles collection with all four settings listed and targeting configured]`

Allow a few minutes for the profiles to push via MDM. Monitor progress on the collection's **Overview** tab — all four settings should reach **Installed** before proceeding to Step 5.

![](https://files.readme.io/b7d91cbb1dc74361cf1e3461de29eaa0495b4d3bc1417fea37d81b098de14eeb-image.png)

`[SCREENSHOT: Device Management collection Overview tab showing all four Origin settings with Installed status at 100%]`

---

## 

Step 4 — Verify Profiles on the Endpoint

Before building the Deploy package, confirm the profiles are installed correctly on a test endpoint.

**All four profiles installed:**

Shell

```
sudo profiles show -all | grep -i "origin"
```

Should return entries for all four profiles.

**System extension loaded:**

Shell

```
systemextensionsctl list | grep com.origin.dns-proxy
```

Status should be `[activated enabled]`.

**In Tanium Interact**, confirm the sensor output looks correct — run this question against your test Mac:

```
Get Device Management - Settings Installation Status from all machines with Computer Name equals <your-test-mac>
```

You should see output like:

```
Origin - Privacy | Installed | Installation successful | <timestamp>
Origin - Proxy Root CA | Installed | Installation successful | <timestamp>
Origin - System Extensions | Installed | Installation successful | <timestamp>
Origin - VPN | Installed | Installation successful | <timestamp>
```

![](https://files.readme.io/7b08089f23da618ae0c2e9d16edb8c50f83596aaf50c70fe3fddcbacf6b9783f-image.png)

`[SCREENSHOT: Tanium Interact showing Device Management - Settings Installation Status results for the test Mac with all four Origin profiles showing Installed]`

---

## 

Step 5 — Create the Computer Group

The Deploy deployment is scoped to this computer group. A Mac joins the group only when all four Origin profiles are confirmed installed — this is the gate that prevents the Deploy package from running on unprepared endpoints.

1. Go to **Administration → Computer Groups**
2. Click **New Computer Group**
3. Name it `Origin Profiles Installed`
4. Set the filter expression to:

```
( Is Mac equals true 
  and ( Device Management - Settings Installation Status matches "^Origin - Privacy\s*\|\s*Installed.*$"
    and Device Management - Settings Installation Status matches "^Origin - Proxy Root CA\s*\|\s*Installed.*$"
    and Device Management - Settings Installation Status matches "^Origin - System Extensions\s*\|\s*Installed.*$"
    and Device Management - Settings Installation Status matches "^Origin - VPN\s*\|\s*Installed.*$"
  )
)
```

5. Click **Preview** to confirm your test Mac appears in the group
6. Click **Save**

![](https://files.readme.io/5aee82b99d2a9ec8d9341a7fe0f52095cdd8105f80219704917bb03f268638e3-image.png)

`[SCREENSHOT: Computer Group editor showing the Origin Profiles Installed group with all four regex conditions configured]`

![](https://files.readme.io/cf176fded6f7455e879e8d0eb2bc9c349f8d559972087606134e22ec76b0fea1-image.png)

`[SCREENSHOT: Computer Group preview showing the test Mac listed as a member]`

**How the regex works:** Each condition matches a row in the sensor output that begins with the specific profile name, followed by a pipe delimiter (with optional whitespace), followed by `Installed`. This ensures both the profile name AND its status are confirmed together — a condition like `contains "Installed"` alone would match any profile that happens to be installed, not the specific one named.

---

## 

Step 6 — Create the Deploy Package

A single Deploy software package with three sequential commands handles JWT staging, installation, and cleanup.

### 

Package details

1. Go to `Modules → Deploy → Software → Software Packages`
2. Click **Create Package**
3. Fill in the package details:

| Field | Value |
| --- | --- |
| Name | `Origin Agent - Install (macOS)` |
| Vendor | `Prelude Research` |
| Version | `1.0.16` |
| Platform | `macOS` |

### 

Package files

In the **Package Files** section, upload both files:

- `Origin.pkg`
- `jwt.txt` — a plain text file containing your provisioning JWT (just the token string, no newline)

`jwt.txt` is uploaded as a package file so Deploy distributes it to the endpoint before any commands run. The JWT never appears in the command body and is therefore not visible to admins browsing package details.

![](https://files.readme.io/2faaca1cacd4bdd32a9515c3c27ef2a1752811a84303227dde46c3bbf0d76ba7-image.png)

`[SCREENSHOT: Deploy package editor showing Package Files section with both Origin.pkg and jwt.txt uploaded]`

### 

System requirements

- **Architecture:** x64
- **Minimum OS:** macOS 11.0

### 

Install operation

Enable the **Install** operation and ensure **Require Source Files** is selected. Add three sequential Run Commands:

**Command 1 — Stage the JWT**

Shell

```
install -m 400 jwt.txt /var/tmp/origin-provisioning-jwt
```

This copies `jwt.txt` to the path the postinstall expects, with permissions set to read-only by owner (root/SYSTEM). Success code: `0`.

**Command 2 — Install Origin**

Shell

```
installer -pkg Origin.pkg -target /
```

Success code: `0`.

**Command 3 — Clean up the JWT**

Shell

```
rm -f /var/tmp/origin-provisioning-jwt
```

Success code: `0`.

If Command 1 or 2 fails, Deploy will not proceed to the next command. The cleanup in Command 3 also runs on success — if the installer itself fails and leaves the JWT file behind, note that the file has mode 400 (root read-only) so it is not readable by other processes.

![](https://files.readme.io/870913b8670632e99dc40b7a094d225929adb2086191e1dfe2b08282ff25e296-image.png)

`[SCREENSHOT: Deploy package editor showing the Install operation with all three commands listed in order]`

### 

Installation requirements

This ensures the package only runs on Macs that do **not** already have Origin installed:

| Attribute | Operator | Value |
| --- | --- | --- |
| File Path | does not exist | `/Applications/Origin.app` |

### 

Install verification

This confirms Origin installed successfully:

| Attribute | Operator | Value |
| --- | --- | --- |
| File Path | exists | `/Applications/Origin.app` |

![](https://files.readme.io/3806feb647710e79b7c142f07f8d5b494f0f625250949eca516cd7aaf689b13a-image.png)

`[SCREENSHOT: Deploy package editor showing Installation Requirements and Install Verification sections]`

### 

Save the package

Click **Create Package** and wait for **Status** to reach **100%** before deploying.

---

## 

Step 7 — Create the Deployment

1. Go to `Modules → Deploy → Deployments` and click **New Deployment**
2. Select **Software Package** and choose `Origin Agent - Install (macOS)`
3. Select **Install** as the operation
4. In **Endpoints to Target**, select the `Origin Profiles Installed` computer group
5. Set **Deployment Type** to **Ongoing** — this ensures newly enrolled Macs automatically receive the agent once their profiles reach Installed status
6. Configure a maintenance window if required by your change management policy
7. Click **Deploy**

![](https://files.readme.io/79919286430bae14b5bf8280034224211ae2eeb0ad2f04479206672408b1a2bb-image.png)

`[SCREENSHOT: Deploy new deployment wizard showing Origin Agent - Install (macOS) package, Install operation, and Origin Profiles Installed computer group selected]`

**The ongoing deployment plus the dynamic computer group makes this fully zero-touch for new Macs going forward.** A Mac enrolls in Device Management → profiles push via MDM → profiles reach Installed → Mac joins the `Origin Profiles Installed` group → Deploy fires automatically.

---

## 

Step 8 — Verification

Run these checks on a target endpoint after the deployment completes.

### 

Deploy job status

Open the deployment in `Modules → Deploy → Deployments` and confirm the endpoint shows **Installed** status.

![](https://files.readme.io/574cba54102006d503c5a4106ab51b9f46e46ba66b72d343e75d490d2ad82d1b-image.png)

`SCREENSHOT: Deploy deployment status view showing target Mac with Installed status]`

### 

Profiles installed

Shell

```
sudo profiles show -all | grep -i "origin"
```

Should return all four Origin profiles.

### 

System extension loaded

Shell

```
systemextensionsctl list | grep com.origin.dns-proxy
```

Expected: `[activated enabled]`

### 

Agent process running

Shell

```
pgrep -x origin
```

Returns a PID when the agent is running.

### 

JWT cleaned up

Shell

```
ls /var/tmp/origin-provisioning-jwt
```

Expected: `No such file or directory`

### 

Agent log

Shell

```
cat /Library/Origin/logs/agent.log.$(date -u +%Y-%m-%d)
```

Confirms agent started cleanly and registration succeeded.

### 

Origin console

In the Origin console, click **Computers** and search for the target hostname. A successfully registered endpoint will appear with a recent snapshot time and its detected AI agents populated.

![](https://files.readme.io/a256f721bd60570a2a7f2b4c880ad98a9b01e447f3637e6b51ed71414eb34ef0-image.png)

`[SCREENSHOT: Origin console Computers view showing the enrolled Mac with recent snapshot time and AI agents detected]`

---

## 

Step 9 — Troubleshooting

### 

Profile shows Failed or Pending in Device Management

Check the setting status on the collection's Devices tab.

- **AllowedTeamIdentifiers conflict** — the profile XML was modified or recreated using a payload editor. Re-upload the raw `.mobileconfig` from the deployment package.
- **Profile upload returns 500 / "missing required information"** — check that `PayloadCertificateFileName` is present in the proxy CA profile inner payload dict (see Step 2).
- **Profile stuck Pending** — trigger an MDM check-in from the device's Endpoint Details page using **Deploy Action → Refresh Device Data**, or wait for the next scheduled check-in.

### 

Mac not appearing in the `Origin Profiles Installed` computer group

Run the sensor question manually in Interact to confirm the output format matches the regex:

```
Get Device Management - Settings Installation Status from all machines with Computer Name equals <hostname>
```

If the profile names in the output differ from `Origin - Privacy`, `Origin - Proxy Root CA`, `Origin - System Extensions`, `Origin - VPN` — update the computer group regex to match the actual names.

### 

JWT prompt appears during install (non-silent install)

The installer postinstall did not find the staged JWT. Check:

1. Command 1 (`install -m 400 jwt.txt /var/tmp/origin-provisioning-jwt`) completed with exit code 0 — check the Deploy job log
2. `jwt.txt` was uploaded to the package files and **Require Source Files** is enabled on the Install operation
3. `jwt.txt` contains only the JWT string with no trailing newline or whitespace

### 

Deploy package shows Not Applicable

The Installation Requirements rule (`/Applications/Origin.app` does not exist) is evaluating as not met — meaning Origin is already installed on the endpoint. Check the endpoint in the Origin console to confirm whether it is already registered.

### 

Origin not appearing in Origin console after install

- Confirm outbound HTTPS (port 443) from the endpoint to the Origin cloud backend is not blocked by proxy or firewall
- Check the agent log: `cat /Library/Origin/logs/agent.log.$(date -u +%Y-%m-%d)`
- Confirm the JWT in `jwt.txt` was valid and had not expired at install time

---

## 

Step 10 — Maintenance

### 

Updating the Origin agent

1. Download the new `Origin.pkg` from the Origin console
2. Edit the `Origin Agent - Install (macOS)` package in Deploy — replace `Origin.pkg` in Package Files and increment the version
3. Update the **Install Verification** if the app version changes
4. The ongoing deployment automatically pushes the update to enrolled endpoints

### 

Rotating the provisioning JWT

1. Obtain the new JWT from the Origin console under `Settings → Installers`
2. Create a new `jwt.txt` containing the new token
3. Edit the Deploy package — replace `jwt.txt` in Package Files and increment the version
4. Existing registered agents are not affected — they maintain their own session credentials after initial registration. Only endpoints receiving a fresh install will use the updated JWT.

### 

Removing the agent

A `Resources/uninstall.sh` script is bundled inside the app:

Shell

```
/Applications/Origin.app/Contents/Resources/uninstall.sh
```

Add a **Remove** operation to the Deploy package using this command and deploy to the target computer group.

### 

Mac Device Enrollment deprecation

Tanium Mac Device Enrollment is deprecated as of December 2025 and sunsets September 2026. If your environment currently uses Mac Device Enrollment, migrate to Tanium Device Management before implementing this guide. See the _Tanium Endpoint Management for Mobile User Guide_ for migration steps.

---

_ORIGIN · PRELUDE RESEARCH, INC. · ALL RIGHTS RESERVED · TANIUM GUIDE_

Copy Page