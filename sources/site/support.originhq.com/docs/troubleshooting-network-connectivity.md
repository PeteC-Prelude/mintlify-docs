# Source: https://support.originhq.com/docs/troubleshooting-network-connectivity

# 

Origin Agent — Network Connectivity Requirements

The Origin agent depends on outbound access to a small set of Prelude endpoints. This document lists those endpoints and shows how to verify reachability on Windows and macOS. Additionally if these endpoints are unavailable during install, the install will fail.

---

## 

Required endpoints

The Origin agent communicates with two Prelude-hosted endpoints, both of which are reached over standard HTTPS on TCP port 443.

| Endpoint | Used during | Purpose |
| --- | --- | --- |
| `registration.prod.originhq.com` | Installation | Endpoint registration. The installer exchanges the provisioning token for the agent's unique client certificate. Reached once, at install time. |
| `endpoint.prod.originhq.com` | Ongoing operation | The agent's steady-state connection. Used for heartbeat, configuration retrieval, and reporting of observed AI activity. |

> **If a web proxy or SASE gateway is in the path:** Both hostnames must be allowed on TCP 443. They should also be **excluded from TLS/SSL inspection** — the agent validates the certificate it receives, so a gateway that re-signs the connection will cause registration and heartbeat to fail.

---

## 

Verifying on Windows

Run the following in PowerShell on the target endpoint. This confirms the endpoint can open a TCP connection to each Prelude host on port 443.

PowerShell

```
Test-NetConnection registration.prod.originhq.com -Port 443
Test-NetConnection endpoint.prod.originhq.com -Port 443
```

> **Expected output:** For both commands, `TcpTestSucceeded` should report `True`. A populated `RemoteAddress` also confirms the hostname resolved in DNS.

Both endpoints speak gRPC/HTTPS rather than serving a web page, so a successful TCP connection on 443 is the meaningful signal — an HTTP error returned by a browser or by `Invoke-WebRequest` against the bare hostname is expected and is not itself a failure. To additionally confirm the TLS handshake completes end to end (useful when a proxy sits in the path), run:

PowerShell

```
curl.exe -sv https://registration.prod.originhq.com 2>&1 | Select-String "Connected|SSL|TLS|certificate"
```

A clean handshake shows the connection being established and the TLS certificate being presented without error.

---

## 

Verifying on macOS

Run the following in Terminal on the target endpoint. `nc` tests TCP reachability to each Prelude host on port 443.

Bash

```
nc -vz registration.prod.originhq.com 443
nc -vz endpoint.prod.originhq.com 443
```

> **Expected output:** Each command should report `Connection to ... port 443 [tcp/https] succeeded!`

To confirm the TLS handshake completes — and to see whether a proxy is intercepting the connection — run:

Bash

```
curl -sv https://registration.prod.originhq.com 2>&1 | grep -iE "connected|SSL|TLS|certificate"
```

If DNS resolution itself is in question, confirm the hostname resolves before testing the connection:

Bash

```
dig +short registration.prod.originhq.com
dig +short endpoint.prod.originhq.com
```

---

## 

If connectivity is unavailable

Installation happens in two main phases. The first phase is copying files and registering the agent's service or daemon and does not require network access. The second phase, backend registration against `registration.prod.originhq.com` does. When that endpoint is unreachable, the install fails and does not complete successfully.

### 

What the failure looks like

- **Windows:** the MSI's final registration step runs during `InstallFinalize`. If it cannot reach the registration endpoint, it returns a non-zero result and the installer reports a failed install. Under Intune, this surfaces as a failed app installation.
- **macOS:** the `.pkg` postinstall script performs registration. If the endpoint is unreachable, the postinstall step fails and the package install reports failure.

In both cases the agent never receives its client certificate, so it cannot authenticate to Prelude even if connectivity is restored later — the install needs to be re-run once the endpoint is reachable.

### 

If registration succeeds but the steady-state endpoint is later blocked

If `registration.prod.originhq.com` was reachable at install time but `endpoint.prod.originhq.com` is later blocked — for example by a subsequent firewall or proxy change — the agent stays installed and registered but cannot send heartbeats, retrieve configuration, or report AI activity. In the Origin console the endpoint will show a stale or missing "last seen" time and will report no telemetry.

To avoid both failure modes, verify that both endpoints are reachable using the checks in the second and third section before deploying the agent at scale.

---

 

Copy Page