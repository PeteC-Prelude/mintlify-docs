# Source: https://support.originhq.com/docs/origin-agent-fails-to-install-error-1073741515

# 

Origin agent fails to install — error -1073741515 (Undefined)

When installing the Origin agent on Windows, the installer exits with:

```
Error: -1073741515 Undefined
```

This is the decimal form of `0xC0000135` (`STATUS_DLL_NOT_FOUND`). It indicates that a required DLL — specifically one provided by the **Microsoft Visual C++ Redistributable** — is missing from the system. The Origin agent depends on the VC++ runtime, and if the redistributable is not present the agent's binaries cannot load and the install fails.

---

## 

Resolution

Install the latest Microsoft Visual C++ Redistributable **before** running the Origin agent installer.

Download the latest supported package directly from Microsoft:

- **Microsoft documentation:** [Latest supported Visual C++ Redistributable downloads](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)

Direct download links (Visual Studio 2015–2022 runtime):

| Architecture | Download |
| --- | --- |
| x64 | [vc\_redist.x64.exe](https://aka.ms/vs/17/release/vc_redist.x64.exe) |
| x86 | [vc\_redist.x86.exe](https://aka.ms/vs/17/release/vc_redist.x86.exe) |
| ARM64 | [vc\_redist.arm64.exe](https://aka.ms/vs/17/release/vc_redist.arm64.exe) |

> Most Windows endpoints require the **x64** package. On ARM64 hardware (e.g. Surface Pro X or Snapdragon-based devices) install the ARM64 package instead.

After the VC++ Redistributable installs successfully, re-run the Origin agent installer. Installation should complete normally.

---

## 

Treat the VC++ Redistributable as a prerequisite

For broad deployments via Intune, SCCM/MECM, or any other endpoint management platform, **chain or bundle the Visual C++ Redistributable as a prerequisite** so it is installed before the Origin agent. Doing so prevents this failure from occurring on endpoints where the runtime is not already present — commonly fresh OS images, minimal builds, and Windows Server Core installations.

---

 

Copy Page