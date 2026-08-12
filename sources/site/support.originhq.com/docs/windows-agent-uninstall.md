# Source: https://support.originhq.com/docs/windows-agent-uninstall

To uninstall the Origin agent from a Windows endpoint, run `msiexec.exe` with the `/X` (uninstall) switch, pointing it at the same MSI that was used to install the agent. The installer filename follows the pattern `origin-installer-<version>-windows-x64.msi`:

bat

```
msiexec.exe /X origin-installer-x.y.z-windows-x64.msi /quiet /norestart
```

## 

Remove the Origin data folder

The uninstaller does not remove the Origin data directory. After the uninstall completes, delete it manually to fully clean up the endpoint:

```
C:\ProgramData\Origin
```

Copy Page