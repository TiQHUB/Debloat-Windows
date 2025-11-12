# Debloat-Windows
This PowerShell script is designed to remove unnecessary Windows pre-installed applications (bloatware), disable telemetry, and clean up the Windows environment to improve system performance and privacy.

## Features
- Uninstalls common OEM and Microsoft Store bloatware apps such as Xbox, OneDrive, Cortana, and others.
- Removes associated registry keys related to the removed apps.
- Disables telemetry and feedback components to enhance user privacy.
- Cleans up scheduled tasks and disables intrusive features like live tiles.
- Makes Windows leaner and faster by decluttering the system.

## Usage
- Run the script with administrator privileges to ensure it has the necessary permissions to uninstall apps and modify system settings.
- The script is designed to be run in a PowerShell window with execution policy bypassed, for example:
```powershell
powershell.exe -ExecutionPolicy Bypass -File RemoveBloat.ps1
```
The script may run silently or prompt for some inputs depending on its version.


## Debloat-Windows Implementation using Intune
