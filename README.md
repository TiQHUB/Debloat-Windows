# Windows 10 / 11 Enterprise Debloat Script

**Current version:** 5.6.5
**Based on:** the De-Bloat script by [Andrew Taylor](https://andrewstaylor.com)

## Overview

`DebloatWindows.ps1` is a Windows debloating and privacy-hardening script for fresh Windows 10 and Windows 11 builds, including Autopilot, Intune and Windows Autopilot Device Preparation deployments.

The script removes:

- OEM software from HP, Dell, Lenovo, Samsung, Acer and Asus
- Consumer AppX apps and their provisioned packages
- McAfee, using the McAfee Consumer Product Removal tool
- Retail Microsoft 365 / OneNote and Google Chrome (during OOBE only)
- Copilot, Recall, Cortana, Spotlight, Feeds, and Xbox / gaming components
- The OneDrive Photos app and the Edge Surf game

It also:

- Applies privacy settings to every existing user profile and to the Default user profile
- Optionally resets the Start menu layout
- Supports a custom whitelist and a custom removal list
- Supports removal of custom scheduled tasks
- Detects OOBE and Intune state to avoid removing user- or Intune-installed apps
- Logs every action for auditing

## Repository Contents

| File | Purpose |
| --- | --- |
| `DebloatWindows.ps1` | Main debloat script (code-signed) |
| `DebloatWindows.zip` | Packaged copy of `DebloatWindows.ps1`, downloaded by the Intune wrapper |
| `debloat-intune-script.ps1` | Intune / RMM wrapper that downloads the zip, extracts it and runs the script with parameters |

## Key Features

- **Manufacturer detection:** runs HP, Dell, Lenovo, Samsung, Acer or Asus cleanup based on the detected device manufacturer
- **AppX removal:** removes both installed AppX packages and provisioned packages
- **Privacy hardening:** disables Cortana, Copilot, Recall, consumer experiences and content delivery
- **Gaming components:** disables Xbox services, Game Bar and GameDVR
- **All-user coverage:** applies settings to every user profile and the Default user hive, so new users inherit them
- **Start menu reset:** optional, and safe for Autopilot pre-provisioning
- **Process safety:** checks whether a process is marked critical before stopping it, to avoid `CRITICAL_PROCESS_DIED` stop errors
- **Silent uninstalls:** handles MSI, Burn bundle, InstallShield and NSIS uninstallers
- **Logging:** full transcript including total run time
- **Customization:** whitelist, removal list and scheduled task parameters

## Supported Operating Systems

| Operating System | Supported |
| --- | --- |
| Windows 10 | Yes |
| Windows 11 | Yes |

## Logging and Output

| Location | Description |
| --- | --- |
| `C:\ProgramData\Debloat\Debloat.log` | Full execution transcript, including total run time |
| `C:\ProgramData\Debloat\` | Working folder for downloaded tools and the Office Deployment Tool configuration |
| Console output | Real-time progress |

## Parameters

```powershell
param (
    [string[]]$customwhitelist,
    [string[]]$TasksToRemove,
    [string[]]$custombloatlist,
    [string]$AllowStartMenuReset = "true"
)
```

| Parameter | Default | Description |
| --- | --- | --- |
| `customwhitelist` | None | Apps that must not be removed. Added to the built-in whitelist and also respected by the manufacturer sections. |
| `custombloatlist` | None | Additional AppX apps to remove. Wildcards are supported. Overrides any matching whitelist entry. |
| `TasksToRemove` | None | Names of scheduled tasks to disable and unregister. |
| `AllowStartMenuReset` | `"true"` | Set to any value other than `"true"` to skip the Start menu reset. |

List parameters accept either an array or a single comma-separated string.

## Usage

The script elevates itself if it is not run as Administrator, and passes all parameters through to the elevated session.

### Default run

```powershell
.\DebloatWindows.ps1
```

### Keep specific apps

```powershell
.\DebloatWindows.ps1 -customwhitelist "Microsoft.Paint","Microsoft.WindowsCalculator"
```

### Remove additional apps

```powershell
.\DebloatWindows.ps1 -custombloatlist "Microsoft.YourPhone","Microsoft.Todos"
```

### Remove custom scheduled tasks

```powershell
.\DebloatWindows.ps1 -TasksToRemove "OfficeTelemetryAgentFallBack","OfficeTelemetryAgentLogOn"
```

### Skip the Start menu reset

```powershell
.\DebloatWindows.ps1 -AllowStartMenuReset "false"
```

### Combined

```powershell
.\DebloatWindows.ps1 `
  -customwhitelist "Microsoft.Paint","Microsoft.WindowsCalculator" `
  -custombloatlist "Microsoft.YourPhone" `
  -TasksToRemove "OfficeTelemetryAgentFallBack" `
  -AllowStartMenuReset "false"
```

## Deploying with Intune

`debloat-intune-script.ps1` is a lightweight wrapper for Intune (platform script or Win32 app) or any RMM tool. It:

1. Creates `C:\ProgramData\Debloat`
2. Downloads `DebloatWindows.zip` from the `main` branch of this repository
3. Extracts the zip and runs `DebloatWindows.ps1` with the configured parameters

Set these variables at the top of the wrapper before deploying:

```powershell
# Define apps to whitelist (comma-separated)
$whitelistApps = ""

# Define apps to bloat (comma-separated)
$bloatApps = ""

# Define scheduled tasks to remove (empty by default). Comma separated
$tasksToRemove = @() -join ','

# Allow start menu reset
$startMenuReset = $true
```

Run the wrapper in the SYSTEM context using 64-bit PowerShell. In the SYSTEM context, per-user settings are applied through `HKU\<SID>` and the Default user hive rather than `HKCU`.

**Note:** after changing `DebloatWindows.ps1`, rebuild `DebloatWindows.zip`. The wrapper only downloads the zip.

## What the Script Does

### Application Removal

- Removes consumer AppX apps and their provisioned packages, including Bing News and Weather, Solitaire, Xbox apps, Spotify, Disney+, Dropbox OEM, Microsoft Family, Quick Assist, Dev Home, Feedback Hub, Maps, and Mail and Calendar.
- Never removes language packs, speech and voice packs, or the built-in list of non-removable system apps.
- Keeps a default whitelist: Notepad, Calculator, Photos, Paint, Snipping Tool, Camera, Sticky Notes, Microsoft Store, Windows Terminal, Company Portal, the new Outlook, Teams, Edge, Windows Security, media codecs and extensions, App Installer, and selected Dell utilities.
- Does not remove the Microsoft 365 app (Office Hub), because removing it also removes Microsoft 365 Copilot.
- Removes OEM-installed Win32 apps using their registered uninstallers:
  - MSI: `msiexec /X {GUID} /quiet /norestart /qn`
  - EXE: uses `QuietUninstallString` when available; otherwise adds the correct silent switch for Burn bundles (`/quiet`), the InstallShield SupportAssist uninstaller (`/arp /S /v/qn`) and NSIS (`/S`)
  - Handles unquoted uninstall paths that contain spaces (common with HP software)
  - Waits for each uninstaller to finish and logs its exit code
- Removes McAfee completely: runs the McAfee Consumer Product Removal tool, uninstalls remaining McAfee products, Safe Connect and WebAdvisor, and cleans up leftover Start menu, registry and AppX entries.
- Removes the OneDrive Photos app (`OneDrive.App.exe`) and its Start menu shortcuts.
- On Windows 10, removes the Windows Backup app and then restores Shell Components so the Snipping Tool continues to work.

### Manufacturer-Specific Cleanup

| Manufacturer | Examples of software removed |
| --- | --- |
| HP | Wolf Security (removed in the required order), Sure Recover, Sure Run, Insights and Analytics, myHP, Support Assistant, Performance Advisor, Presence Video, Poly Lens, Connect Optimizer, HP Documentation |
| Dell | SupportAssist (including Remediation and OS Recovery), Optimizer, Command Update, Power Manager, Digital Delivery, Pair, Display Manager, Core Services, Trusted Device |
| Lenovo | Vantage (skipped if Commercial Vantage is installed), AI Now, Smart Appearance, Smart Meetings, Smart Microphone Settings, Smart Storage, TrackPoint Quick Menu, Lenovo Now, Lenovo Welcome, User Guide, X-Rite Color Assistant |
| Samsung | Bixby, Galaxy Book Experience, Galaxy Book Smart Switch, Samsung Account, Cloud, Notes, Gallery and Flow, Device Care, Quick Share, SmartThings, Live Wallpaper |
| Acer | Jumpstart, Quick Access, ProShield Plus, Configuration Manager, Purified Voice Console, Product Registration, User Experience Improvement Program service, bundled McAfee, Evernote and Dropbox |
| Asus | MyASUS, Expert Widget, Intel Graphics and Intel Security utilities, Dolby, Dirac audio, AI meeting tools, OEM theme, wallpaper and taskbar layout |

Running OEM processes and services are stopped before uninstalling so their files are not locked. Processes that Windows marks as critical are never stopped.

### Privacy and Security Settings

- Disables Cortana, including search integration, inking and typing personalization, and contact harvesting
- Disables Copilot on Windows 11 for the machine, all users, the `.DEFAULT` profile and the Default user hive
- Disables Recall (`DisableAIDataAnalysis`)
- Disables Windows Spotlight on the lock screen and desktop
- Disables consumer experiences and silent, OEM and suggested app installs
- Disables Bing web search and game results in Windows Search
- Disables the advertising ID and Windows feedback prompts
- Disables Wi-Fi Sense
- Disables live tiles and the People taskbar icon
- Hides the "Learn about this picture" desktop icon
- Removes the 3D Objects folder from This PC
- Skips unnecessary OOBE screens (privacy, voice, EULA and first sign-in animation) for Device Preparation

**Note:** telemetry and location services are intentionally left enabled, because Intune, Endpoint Analytics reporting and Find My Device depend on them. The code to disable them is included in the script, commented out, for environments that do not use Intune.

### Gaming and Consumer Features

- Disables Xbox services: `XblAuthManager`, `XblGameSave`, `XboxGipSvc`, `XboxNetApiSvc` and `xbgm`
- Disables Xbox and Customer Experience Improvement Program scheduled tasks
- Disables Game Bar and GameDVR capture, which also stops Game Bar popups
- Removes `GameBarPresenceWriter.exe`
- Hides the Gaming pages in Settings
- Disables the Edge Surf game
- Disables Windows Feeds / News and Interests (the Widgets board)

### Start Menu

Controlled by the `AllowStartMenuReset` parameter (default `"true"`).

#### Windows 10

- Replaces `C:\Windows\StartLayout.xml` with a blank layout

#### Windows 11

- Replaces the Default user's pinned apps with a minimal set (Edge, Microsoft Store, File Explorer)
- Deploys a clean `Start2.bin` to the Default profile only if no non-administrator user has signed in yet. This makes it safe during Autopilot pre-provisioning and leaves existing users unaffected.

### Intune and OOBE Awareness

- Uses the Windows `OOBEComplete` API to determine whether the device is still in OOBE
- Removes Google Chrome and retail Microsoft 365 / OneNote (all languages) only during OOBE, using the Office Deployment Tool. Enterprise editions of Office are kept.
- Skips those removals after OOBE, so apps installed by users or Intune are not affected
- Never removes the Intune Management Extension or Company Portal
- Accounts for the `defaultuser0` profile so the Start menu logic works correctly during the Enrollment Status Page

## Design Notes

- Elevates itself when not run as Administrator
- Suppresses progress bars and unnecessary prompts
- Applies user settings through `HKU\<SID>`, so it works in the SYSTEM context
- Checks for critical processes before stopping any process
- Runs uninstalls one at a time and waits for each to complete
- Supports non-English Windows builds (administrator group lookup by SID, localized Office and shortcut cleanup)
- Logs uninstaller exit codes
- Code-signed

## Important Notes

- Restart the device after the script completes.
- Test with a pilot group before deploying widely.
- Internet access is required to download the McAfee removal tool, the Office Deployment Tool and `Start2.bin`, and the zip file when using the Intune wrapper.
- OEM recovery partitions are not removed.
- The script is designed for enterprise environments.
- The full change history is in the `.NOTES` block at the top of `DebloatWindows.ps1`.
