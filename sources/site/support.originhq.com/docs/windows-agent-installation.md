# Source: https://support.originhq.com/docs/windows-agent-installation

The MSI installer is available from the Origin Console. As noted in the prerequisites, this install option requires Microsoft Visual C++ to be installed. An `.exe` installer is also available upon request and will install Microsoft Visual C++ automatically if it is missing.

## 

Silent Install (MSI)

bat

```
msiexec /i origin-installer-x_y_z-windows-x64.msi /quiet PROVISIONING_JWT=<your_jwt_token>
```

## 

Silent Install with Enhanced Logging

bat

```
msiexec /i origin-installer-x_y_z-windows-x64.msi ^
  /quiet ^
  /norestart ^
  /l*v "C:\ProgramData\Origin\Agent\Logs\msi-install.log" ^
  PROVISIONING_JWT=<your_jwt_token>
```

## 

Silent Install (MSI) — OTEL Configuration

For OTEL-based deployments, add `INSTALL_WINDIVERT=0` to the silent install command to skip installing the WinDivert network driver:

bat

```
msiexec /i origin-installer-x_y_z-windows-x64.msi /quiet PROVISIONING_JWT=<your_jwt_token> INSTALL_WINDIVERT=0
```

## 

Log Locations

| Log | Path |
| --- | --- |
| Install Log | `C:\ProgramData\Origin\Agent\Logs\install.log` |
| Detailed MSI Log | `%temp%\Origin_Observability_Agent_xxxx.log` |
| MSI Component Log | `%temp%\Origin_Observability_Agent_xxxx_OriginAgentMsi.log` |
| Agent Runtime Logs | `C:\ProgramData\Origin\Agent\Logs\` |

## 

System Tray Icon

The Windows Origin install creates a system tray icon that provides visibility into the overall health of the agent, allows you to run a health diagnostic report, and provides quick access to agent logging.

Copy Page