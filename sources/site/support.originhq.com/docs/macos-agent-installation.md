# Source: https://support.originhq.com/docs/macos-agent-installation

This guide walks you through installing the Origin agent on a macOS endpoint, from running the installer to confirming the agent is reporting in. The whole flow takes about five minutes and requires:

- A macOS host you can sign in to as an admin.
- The Origin agent `.pkg` installer for your tenant.
- The **JWT provisioning token** from your Origin dashboard.

> 📘
> 
> ### 
> 
> **Where to get the installer and token**
> 
> Both live in the Origin dashboard under **Settings → Installers**. Download the latest `.pkg` and copy the JWT to your clipboard before you start.

---

## 

Step 1 — Launch the installer

Double-click the `.pkg` you downloaded. macOS opens the standard Installer app.

![Installer welcome screen](https://files.readme.io/87c3ab41fdab192c979fa6932f70475c761ed6da295d1539993427a58202d311-image7.png)

Click **Continue**.

---

## 

Step 2 — Confirm the install location

The installer shows the disk it will install to (typically _Macintosh HD_) and the disk-space footprint (~89 MB).

![Standard install on Macintosh HD](https://files.readme.io/71dcb1177790802da627f6ff569e218879d690965b2352bc5eb5a174cfe571fe-image4.png)

Click **Install**.

---

## 

Step 3 — Authenticate to install

macOS prompts for your admin password to allow the installer to write to system locations.

![Installer password prompt](https://files.readme.io/c3d47c5040dec6839dd5f0371a6a8291ce8b68f8745ef1f51b04826241f8ead5-image10.png)

Enter your password and click **Install Software**.

---

## 

Step 4 — Paste the registration token

After the package extracts, the **Origin Agent Registration** dialog appears. Paste the JWT token you copied from the dashboard.

![Origin Agent Registration prompt](https://files.readme.io/50de7b0100a22180f9fa0d5c28b02a47aed18272c4377307ad53546b05719d9d-image8.png)

> ⚠️
> 
> ### 
> 
> **Important:** `⌘ + V` does **not** work in this prompt field. **Right-click → Paste** instead. (The field intentionally blocks the keyboard paste shortcut to prevent it from being filled by stray clipboards.)

Click **Register**.

---

## 

Step 5 — Approve certificate trust changes

The agent installs a trust anchor so it can intercept and inspect TLS traffic. macOS asks you to re-authenticate before changing System Certificate Trust Settings.

![Certificate trust settings password prompt](https://files.readme.io/d3e449f71e13b2bf3c00f04dad745707e931bcc72991757a166443fb92a995d2-image6.png)

Enter your password and click **Update Settings**.

---

## 

Step 6 — Allow the network extension

The agent ships its traffic-inspection functionality as a macOS **Network Extension**. The first time it tries to load, macOS shows a sheet asking you to enable it in System Settings.

![Origin Activator network extension prompt](https://files.readme.io/e148d3cad613f484fedb4b57015c871b21be85bc5b96aa43b87bad86e533cf13-image11.png)

Click **Open System Settings**. (If you click **OK** by accident, open **System Settings → General → Login Items & Extensions → Network Extensions** manually.)

---

## 

Step 7 — Enable the Origin Activator extension

In the **Network Extensions** panel you'll see _Origin Activator — Origin Network Extension_ with the toggle off.

![Network Extensions panel — Origin Activator disabled](https://files.readme.io/c658da0da139f2ef4745d82da93b443c1d37d639406efe625d11b17e106f4f80-image5.png)

Click the toggle. macOS prompts for your password to allow the system extension change.

![System Extensions password prompt](https://files.readme.io/ead0175179590534dee608dda0fb577a5b3d76e64f0b38a182790d386669635c-image2.png)

Enter your password and click **OK**. The toggle should now show enabled.

![Network Extensions panel — Origin Activator enabled](https://files.readme.io/a5c8bd5fa2b44cfba310ad332b5c643c4d2af70c0e11006ba72bc2e6c9b54aed-image13.png)

Click **Done**.

---

## 

Step 8 — Allow the installer to control System Events

The installer needs to drive **System Events** to finish wiring the agent into the OS.

![Installer wants access to control System Events](https://files.readme.io/40e950b53d83c3af848bd96a6720c16f04a961f8ae9a61a6329e3890f74b7e6b-image9.png)

Click **Allow**.

---

## 

Step 9 — Approve the proxy configuration

The agent registers a transparent proxy so it can see network traffic from the host's apps. macOS surfaces this as a one-time approval.

![Origin would like to add proxy configurations](https://files.readme.io/2591c1c6a2b84e34eb22ae37666c1cb7489ec4cf3b764759f66615f165d08181-image12.png)

Click **Allow**.

> 📘
> 
> ### 
> 
> If you click **Don't Allow** by mistake, you can re-approve in **System Settings → Network → Filters & Proxies**.

---

## 

Step 10 — Grant Full Disk Access

The agent finally prompts you to grant **Full Disk Access** so it can read application logs and detect AI tooling installed in user-scoped locations.

![Full Disk Access required dialog](https://files.readme.io/849edba494ecbb68587b5aed3f59baf42926982cbde0a19f0297aeacc2213978-image14.png)

Click **OK**. macOS opens **System Settings → Privacy & Security → Full Disk Access**.

1. Click the **+** button at the bottom of the list.
2. Navigate to `/Applications` and select **Origin.app**.
3. Click **Open**, then toggle the row on.

![Full Disk Access — Origin enabled](https://files.readme.io/70d765c56d253855babc4db8f6230c799c3090cb3a789aeeef4e03c681c6eb86-image3.png)

The agent will start automatically as soon as the toggle flips on.

---

## 

Step 11 — Confirm background activity

macOS posts a notification confirming the agent is allowed to run in the background.

![App background activity confirmation](https://files.readme.io/0eff0aaf70f5aa4b8f1006404848f9eb0f741e92a12f7cd8a5e5f8faebc8dbde-image1.png)

You're done. The agent is now reporting in. Check the Origin dashboard — the host should appear in the **Endpoint Explorer** within a minute.

---

## 

Verify the install

Open the menu-bar **Origin** icon — a green dot means the agent is healthy. You can also run the built-in health check from there.

If the host doesn't appear in the dashboard within a couple of minutes, see _Log Locations_ below for the install and runtime logs.

---

## 

Log Locations

| Log | Path |
| --- | --- |
| Install & Registration | `/Library/Origin/logs/origin-setup.log` |
| macOS Install Log | `/var/log/install.log` |
| Agent Runtime | `/Library/Origin/logs/origin-agent.log` |

---

## 

Menu Bar Icon

The macOS Origin install creates a menu bar icon. This provides visibility into the overall health of the agent, allows you to run a health diagnostic report, and provides quick access to agent logging.

Copy Page