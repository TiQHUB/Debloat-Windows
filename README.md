# 🧹 Windows 10 / 11 Enterprise Debloat Script

**OEM Bloatware Removal & Privacy Hardening**

---

## 📌 Overview

This PowerShell script is a **comprehensive Windows debloating and privacy-hardening solution** designed for **fresh Windows 10 and Windows 11 builds**.

It removes:

* OEM bloatware (HP, Dell, Lenovo, Samsung)
* Consumer apps and AppX packages
* Telemetry, ads, Copilot, Recall, Cortana
* Xbox, Gaming, Spotlight, Feeds, Widgets
* Scheduled tasks, services, and Edge Surf Game

It also:

* Hardens registry privacy settings
* Clears Start Menu layouts
* Supports **custom whitelisting**
* Supports **custom scheduled task removal**
* Is **Intune & OOBE-aware**
* Logs all actions for auditing

---

## ✨ Key Features

* 🏭 **OEM-aware removal**

  * HP, Dell, Lenovo, Samsung
* 🧼 Removes **AppX + Provisioned Packages**
* 🔐 Disables **Cortana, Copilot, Recall**
* 🧠 Disables **Windows Consumer Experience**
* 🎮 Removes **Xbox & Gaming Services**
* 🌐 Disables **Edge Surf Game**
* 📋 Clears **Start Menu layouts**
* 🧾 Centralized logging
* 🛡 Safe for **Autopilot / Intune / OOBE**
* ⚙ Custom whitelist support
* ⏱ Runtime tracking

---

## 🖥 Supported Operating Systems

| OS                | Supported |
| ----------------- | --------- |
| Windows 10        | ✅        |
| Windows 11        | ✅        |

---

## 📂 Logging & Output

| Path                                 | Description        |
| ------------------------------------ | ------------------ |
| `C:\ProgramData\Debloat\Debloat.log` | Full execution log |
| Console Output                       | Real-time progress |

---

## ⚙️ Script Parameters

```powershell
param (
    [string[]]$customwhitelist,
    [string[]]$TasksToRemove
)
```

### 🔧 Parameters Explained

| Parameter         | Description                      |
| ----------------- | -------------------------------- |
| `customwhitelist` | Apps you **do NOT want removed** |
| `TasksToRemove`   | Custom scheduled tasks to delete |

---

## ▶️ Usage Examples

### Basic Execution

```powershell
.\Debloat-Windows.ps1
```

### With Custom App Whitelist

```powershell
.\Debloat-Windows.ps1 -customwhitelist "Microsoft.Paint","Microsoft.WindowsCalculator"
```

### Remove Custom Scheduled Tasks

```powershell
.\Debloat-Windows.ps1 -TasksToRemove "OfficeTelemetryAgentFallBack","OfficeTelemetryAgentLogOn"
```

### Combined

```powershell
.\Debloat-Windows.ps1 `
  -customwhitelist "Microsoft.Paint","Microsoft.WindowsCalculator" `
  -TasksToRemove "OfficeTelemetryAgentFallBack"
```

---

## 🧠 What This Script Does

### 🧹 Application Removal

* Removes **consumer AppX apps**
* Removes **provisioned packages**
* Removes **OEM-installed Win32 apps**
* Removes **McAfee (full cleanup)**

### 🏭 OEM-Specific Cleanup

| Vendor  | Removed                                  |
| ------- | ---------------------------------------- |
| HP      | Wolf Security, Analytics, Support tools  |
| Dell    | SupportAssist, Optimizer, Command Update |
| Lenovo  | Vantage, AI Now, Smart Appearance        |
| Samsung | Bixby, Galaxy Book services              |

---

## 🔐 Privacy & Security Hardening

* Disable **Cortana**
* Disable **Copilot**
* Disable **Recall**
* Disable **Spotlight**
* Disable **Consumer Experience**
* Disable **Telemetry (optional)**
* Disable **Wi-Fi Sense**
* Disable **Advertising ID**
* Disable **Gaming popups**

---

## 🎮 Gaming & Consumer Features Removed

* Xbox services & scheduled tasks
* GameDVR
* Game Bar
* Edge Surf Game
* Windows Feeds
* Widgets
* Live Tiles

---

## 🪟 Start Menu Management

### Windows 10

* Resets Start Menu to **blank layout**

### Windows 11

* Removes default pinned clutter
* Applies clean default layout
* Safe for Autopilot pre-login state

---

## 🧠 Intune & OOBE Awareness

* Detects **OOBE state**
* Detects **Intune app installs**
* Prevents removal of:

  * Required enterprise apps
  * Intune-deployed software

---

## 🛠 Safety & Design Notes

* ✔ Runs elevated automatically
* ✔ Suppresses unnecessary UI prompts
* ✔ Uses registry-safe operations
* ✔ Avoids breaking Windows core components
* ✔ Extensive logging
* ✔ Certificate-signed script compatible

---

## ⚠️ Important Notes

* 🔁 **Reboot recommended** after execution
* 🧪 Test in pilot group before mass deployment
* 📦 OEM recovery partitions are not removed
* 🔐 Designed for **enterprise environments**


Just say the word 👍
