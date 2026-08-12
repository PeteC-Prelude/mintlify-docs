# Source: https://support.originhq.com/docs/origin-console-setup

## 

Settings

Click the **Settings** button in the left navigation bar to open workspace administration. The Settings panel is organized into sections covering fleet-wide endpoint configuration, custom capture rules, trust management, user management, directory integrations, provisioning tokens, skills, and installers.

![](https://files.readme.io/51da55c25a2036aa81883dc4bc82aeae9000faa4f5ec4b2e91bbd944492d5530-Origin-settings-june1.png)

### 

Fleet — Endpoint Configuration

Network proxy settings pushed to every endpoint in the tenant. Changes apply on the next agent heartbeat.

| Toggle | Default | Description |
| --- | --- | --- |
| **Inject RST on missed flow** | On | Resets AI-tool TCP flows that bypassed interception. |
| **Block QUIC (UDP/443)** | On | Forces browsers off QUIC so HTTPS interception works. |
| **Hide system tray icon** | Off | Suppresses the tray process and removes the auto-start entry. |
| **Capture local AI model traffic** | Off | Intercepts on-device LLM runtimes (Ollama, LM Studio, llama.cpp, etc.) on loopback. |

### 

Capture — Custom AI Providers

![](https://files.readme.io/7df470f15f58c99acc886366707ca065213cf0bc705bc671b598b062c8fd29ba-Origin-settings-custom_ai_provider_june1.png)

Capture traffic to AI providers beyond the built-in set. Each rule's domain patterns tell the endpoint proxy which hosts to intercept; captures are attributed under the provider tag. Tenant rules can't shadow built-in providers.

Each rule configures:

- **Rule ID** — a unique identifier for the capture rule
- **Provider tag** — the label applied to captured traffic in the Origin console
- **HTTP method** — the request method to match (e.g. POST)
- **Domain patterns** — bare hostnames with no leading `*.`, matched as exact hostname or subdomain
- **Path contains** — optional substrings that must all appear in the request path (blank matches any path)
- **Capture strategy** — how much of the request to capture; Full request body is the default

### 

Trust — Proxy CA Certificate

![](https://files.readme.io/3cdfa812765c1f10a48bca0ed2fefeada801abd4ad05a09d5bae04b82df51967-Origin-settings-proxyca-june1.png)

The Root CA is distributed via MDM to every endpoint. The certificate's fingerprint, serial number, and issued and expiry dates are displayed here.

Use the action buttons to manage the certificate:

- **Copy PEM** — copy the certificate to the clipboard
- **PEM / DER** — download the certificate in PEM or DER format
- **Rotate CA** — issue a new CA; endpoints pick it up on the next heartbeat and the previous CA is revoked immediately

 

## 

User Management

New users can be invited by clicking the **Settings** button and navigating to **User Management**. Click **Invite User** and provide the email address and role (Member or Admin).

![](https://files.readme.io/11b2b5ca9650ac17d59710505064ff51dbeefd541b3b4f6662478653b57ae3fc-Invite_User.png)

## 

Create a JWT Provisioning Token

A JWT token is required to provision agents against your account/tenant. Click the **Settings** button and navigate to **Provisioning Tokens** to generate one. This token will be used during agent installation on each endpoint. See [Origin Agent Installation](https://support.originhq.com/docs/origin-agent-installation) and [MDM Deployment guides](https://support.originhq.com/docs/mdm-deployment-guides).

![](https://files.readme.io/42faf81a5b9e7997ff7d0bc6b97cc68abd58cda8fbbb5af6b5ef6683aace6ada-JWT.png) 

Copy Page