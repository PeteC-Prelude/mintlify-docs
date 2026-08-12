# Source: https://support.originhq.com/docs/macos-agent-uninstall

To remove the Origin agent from a macOS endpoint, run the bundled uninstall script with administrator privileges:

Bash

```
sudo /Applications/Origin.app/Contents/Resources/uninstall.sh
```

This removes the Origin agent, its network extension, the transparent proxy configuration, and the trust anchor that was installed during setup.

Copy Page