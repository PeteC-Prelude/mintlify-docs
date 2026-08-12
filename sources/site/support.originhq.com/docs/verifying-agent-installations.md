# Source: https://support.originhq.com/docs/verifying-agent-installations

This document details the steps to confirm the Origin Observability Agent is installed, healthy and capturing AI activity on MacOS and Windows Endpoints.

---

## 

Contents

1. [UI and Diagnostic Report](https://support.originhq.com/#1-ui-and-diagnostic-report)
2. [Verify Services and Processes](https://support.originhq.com/#2-verify-services-and-processes)
3. [macOS — verify network proxy](https://support.originhq.com/#3-macos--verify-network-proxy)
4. [Generate Test Activity](https://support.originhq.com/#4-generate-test-activity)
5. [Verify activity in the Origin Platform](https://support.originhq.com/#5-verify-activity-in-the-origin-platform)

---

## 

1\. UI and Diagnostic Report

The Origin Agent runs a small UI accessible from the **system tray** (Windows) or **menu bar** (macOS). The icon color is the fastest health check:

| Icon | Status |
| --- | --- |
| **Green** | Healthy — agent running, registered, capturing |
| **Yellow** | Degraded — running but one or more checks are failing |
| **Red** | Error — agent stopped, unregistered, or interception broken |

Click the icon to open the menu. From here:

- **Diagnostic Report** — runs a full self-test (registration, mTLS credentials, backend reachability, proxy CA, interception, entitlements). Every line should read **OK**.
- **Open Log Folder** — opens the runtime log directory:
 - Windows: `C:\ProgramData\Origin\Agent\Logs\`
 - macOS: `/Library/Origin/logs/`

If the icon is green and the Diagnostic Report comes back clean, the agent is healthy locally.

---

## 

2\. Verify Services and Processes

Both the **Origin Agent** and the **Origin Watchdog** must be running. The watchdog starts at boot and brings up the agent.

### 

Windows (elevated PowerShell)

PowerShell

```
Get-Service OriginWatchdog, OriginAgent
```

Both should report **Running**.

### 

macOS

Bash

```
sudo launchctl list | grep origin
```

Two entries should return: `com.origin.watchdog` and `com.origin.agent`, each with a non-zero PID.

### 

Cross-check in the Origin Platform

Sign in to the **Origin Platform**, open the **Registered Endpoints** view, and confirm your endpoint shows a recent "Snapshot Time". If the endpoint is present here it has registered succesfully with the Origin Platform.

---

## 

3\. macOS — verify network proxy

macOS interception requires a signed DNS proxy system extension. Confirm it is activated:

Bash

```
systemextensionsctl list
```

Look for `com.origin.dns-proxy` in state `[activated enabled]`.

Confirm the proxy CA is in the System keychain:

Bash

```
security find-certificate -a -c "Origin AI Proxy CA" /Library/Keychains/System.keychain
```

One certificate should return, subject `Origin AI Proxy CA`.

End-to-end sanity check:

Bash

```
curl -v https://claude.ai 2>&1 | grep -iE "issuer|subject"
```

If the issuer reads `Origin AI Proxy CA`, interception is working.

---

## 

4\. Generate Test Activity

Pick **1–2** of the following AI generation methods, run a prompt, then validate in the Origin Platform.

> **Tip:** Embed a unique token in each test prompt — e.g. `ORIGIN-VERIFY-<date>` or a unique UUID — to make the prompts trivially searchable.

### 

Sample prompts

A few industry-recognized prompts that exercise different capture archetypes:

- **Summarization** — _"_`ORIGIN-VERIFY-20260513` _— summarize the NIST AI Risk Management Framework (AI RMF 1.0) in three bullets."_
- **Code generation** — _"_`ORIGIN-VERIFY-20260513` _— write a Python function that validates an IPv4 CIDR string."_
- **Rewrite / tone shift** — _"_`ORIGIN-VERIFY-20260513` _— rewrite this for a non-technical executive: \[paste a short paragraph\]."_
- **Open Q&A** — _"_`ORIGIN-VERIFY-20260513` _— list the OWASP Top 10 for LLM Applications."_
- **Data analysis** — _"_`ORIGIN-VERIFY-20260513` _— given this CSV: \[paste 5–10 rows\], identify outliers and explain your reasoning."_

### 

4.1 M365 Copilot — desktop / Office apps

1. Open Word, Excel, or PowerPoint on the verified endpoint.
2. Click the Copilot icon in the ribbon to open the Copilot pane.
3. Send a verification prompt from the list above.
4. Send a follow-up to generate a multi-turn session.

### 

4.2 M365 Copilot — web (`copilot.microsoft.com`)

1. Browse to [https://copilot.microsoft.com](https://copilot.microsoft.com) and sign in.
2. Send a verification prompt.
3. Send a follow-up in the same conversation.

### 

4.3 Direct-to-vendor — Claude.ai / ChatGPT / Gemini

Pick whichever your environment allows: [claude.ai](https://claude.ai), [chatgpt.com](https://chatgpt.com), or [gemini.google.com](https://gemini.google.com).

1. Sign in.
2. Send a verification prompt.
3. Send 2–3 follow-ups to populate a meaningful session.

---

## 

5\. Verify activity in the Origin Platform

Allow 1-2 minutes for events to propagate to the Platform, then:

1. Open the **Origin Platform**.
2. Locate the endpoint in All Endpoints view(this is the default landing page) and click it.
3. Confirm the endpoint's activity bar at the top of the page now shows recent prompt activity — colored ticks indicating prompts captured within the test window.
4. In the **Resource Overview** graph, confirm the agent types you used appear as nodes. Driving Copilot and Claude.ai traffic should populate at least:
 - **ChatGPT** (M365 Copilot's underlying model surface frequently appears under this label)
 - **Claude**
 - **M365 Copilot**

 

Copy Page