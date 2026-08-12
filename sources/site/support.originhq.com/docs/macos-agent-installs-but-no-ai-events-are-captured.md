# Source: https://support.originhq.com/docs/macos-agent-installs-but-no-ai-events-are-captured

# 

macOS agent installs but no AI events are captured

After a manual install of the Origin agent on macOS, the agent appears to be running but no AI activity is reported to the Origin console.

This happens when the **Origin Activator network extension** was not approved during installation. The agent depends on the network extension to observe AI traffic — without approval, the extension is loaded but inactive, so no events are captured.

Manual installs are the common cause: the approval prompt is a one-time interactive step, and if it is dismissed or missed, the extension stays unapproved until a user re-approves it through System Settings.

---

## 

Resolution — approve the network extension

On the affected Mac:

1. Open **System Settings**.
2. Select **General**.
3. Select **Login Items & Extensions**.
4. Scroll down to **Extensions** and switch the view to **By Category**.
5. Click the information icon (ⓘ) next to **Network Extensions**.

![System Settings → General → Login Items & Extensions, with the By Category view selected and the Network Extensions info icon highlighted](https://files.readme.io/09714feead47644b38d657d8a1f6e99ef821c8939c396abf67be32ad1d5cf26a-screenshot_3974.png)

6. Toggle on the **Origin Activator** network extension.
7. Enter the local administrator password when prompted.
8. Click **Done**.

![Network Extensions sheet showing the Origin Activator toggle being enabled](https://files.readme.io/88fed858df3f9152386fc93aa72b833a085a8044c04178b4ec24e43bf67ca3bf-screenshot_3975.png)

The agent begins capturing AI events immediately — no reboot or reinstall is required.

---

## 

Verifying the agent is operational

### 

1\. Check the Origin tray icon

Click the **Origin** icon in the macOS menu bar. In the menu that appears, confirm that **Agent**, **Watchdog**, and **Proxy** each show a **green** status indicator.

![Origin Agent tray menu showing green indicators next to Agent, Watchdog, and Proxy](https://files.readme.io/a701209ec9bf69548f4968ee74a81cd796a29651cf168a9c48f0a38f837b9b1f-screenshot_3977.png)

If any of the three is not green, the corresponding component has not started cleanly — review the agent logs (**Open Logs Folder** in the same menu) before proceeding.

### 

2\. Run Diagnostics

From the same tray menu, select **Run Diagnostics…**. This performs an end-to-end check — including connectivity to the Prelude endpoints and confirmation that the network extension is intercepting traffic — and reports whether the agent is fully operational.

---

 

Copy Page