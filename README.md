# VS Code Multiple Profile Setup Guide

A complete guide to set up separate VS Code installations with different GitHub accounts on Windows.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Creating Custom Icons](#creating-custom-icons)
- [Adding Context Menu Entries](#adding-context-menu-entries)
- [Testing Your Setup](#testing-your-setup)
- [Troubleshooting](#troubleshooting)
- [Removing the Setup](#removing-the-setup)

---

## Overview

This setup allows you to:
- Have completely separate VS Code environments (Work & Personal)
- Use different GitHub accounts for each profile
- Launch profiles from desktop shortcuts or right-click context menu
- Keep extensions, settings, and logins independent

**What gets separated:**
- GitHub account authentication
- Installed extensions
- Editor settings
- Themes and UI customization
- Workspace history

---

## Prerequisites

- Windows 10/11
- VS Code already installed
- Administrator access
- PowerShell

---

## Step-by-Step Setup

### 1. Create Profile Directories

Open **PowerShell** and run:

```powershell
# Create the main directory
mkdir C:\vscode

# Create work profile directory structure
mkdir C:\vscode\work\data

# Create personal profile directory structure
mkdir C:\vscode\personal\data
```

### 2. Copy Your Existing Configuration

Copy your current VS Code settings to both new profiles:

```powershell
# Copy to work profile
xcopy "C:\Users\YOUR_USERNAME\AppData\Roaming\Code" "C:\vscode\work\data" /E /I /H

# Copy to personal profile
xcopy "C:\Users\YOUR_USERNAME\AppData\Roaming\Code" "C:\vscode\personal\data" /E /I /H
```

**Replace `YOUR_USERNAME` with your actual Windows username.**

### 3. Create Launch Scripts

Create two batch files in `C:\vscode\`:

**File: `C:\vscode\vscode-work.bat`**
```batch
@echo off
code --user-data-dir "C:\vscode\work\data" %*
```

**File: `C:\vscode\vscode-personal.bat`**
```batch
@echo off
code --user-data-dir "C:\vscode\personal\data" %*
```

**How to create:**
1. Open Notepad
2. Paste the content
3. File → Save As
4. Change "Save as type" to **All Files**
5. Save with `.bat` extension

### 4. Create Desktop Shortcuts

**For Work Profile:**
1. Right-click Desktop → **New** → **Shortcut**
2. Location: `C:\vscode\vscode-work.bat`
3. Name: `VS Code - Work`
4. Finish

**For Personal Profile:**
1. Right-click Desktop → **New** → **Shortcut**
2. Location: `C:\vscode\vscode-personal.bat`
3. Name: `VS Code - Personal`
4. Finish

### 5. Customize Shortcut Icons (Optional)

**Using VS Code default icon:**
1. Right-click shortcut → **Properties**
2. Click **Change Icon**
3. Browse to: `C:\Program Files\Microsoft VS Code\Code.exe`
4. Select icon → OK

**Using custom icons:**
1. Place your `.ico` files in `C:\vscode\`
2. Right-click shortcut → **Properties**
3. Click **Change Icon**
4. Browse to your `.ico` file
5. OK

---

## Creating Custom Icons

### Converting PNG to ICO

**Option 1: Online Converter (Easiest)**
1. Go to https://convertio.co/png-ico/
2. Upload your PNG file
3. Convert and download

**Option 2: Using IrfanView (Free Software)**
1. Download from https://www.irfanview.com/
2. Install with Plugins
3. Open PNG in IrfanView
4. Image → Resize → 256x256
5. File → Save As → Choose "ICO - Windows Icon"

**Recommendations:**
- Start with square PNG (256x256 minimum)
- Simple designs work best
- Transparent backgrounds are ideal
- Include multiple sizes (16, 32, 48, 256)

### Example Icon Locations
```
C:\vscode\vscode.ico          (Work icon)
C:\vscode\vscode-alt.ico      (Personal icon)
```

---

## Adding Context Menu Entries

This adds "Open with VS Code - Work/Personal" to right-click menus.

### Method 1: Using .reg File (Recommended)

**Create file: `vscode-context-menu.reg`**

```reg
Windows Registry Editor Version 5.00

; Work Profile - Right-click on folder
[HKEY_CLASSES_ROOT\Directory\shell\VSCodeWork]
@="Open with VS Code - Work"
"Icon"="C:\\vscode\\vscode.ico"

[HKEY_CLASSES_ROOT\Directory\shell\VSCodeWork\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" --user-data-dir \"C:\\vscode\\work\\data\" \"%1\""

; Personal Profile - Right-click on folder
[HKEY_CLASSES_ROOT\Directory\shell\VSCodePersonal]
@="Open with VS Code - Personal"
"Icon"="C:\\vscode\\vscode-alt.ico"

[HKEY_CLASSES_ROOT\Directory\shell\VSCodePersonal\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" --user-data-dir \"C:\\vscode\\personal\\data\" \"%1\""

; Work Profile - Right-click inside folder
[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCodeWork]
@="Open with VS Code - Work"
"Icon"="C:\\vscode\\vscode.ico"

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCodeWork\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" --user-data-dir \"C:\\vscode\\work\\data\" \"%V\""

; Personal Profile - Right-click inside folder
[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCodePersonal]
@="Open with VS Code - Personal"
"Icon"="C:\\vscode\\vscode-alt.ico"

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCodePersonal\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" --user-data-dir \"C:\\vscode\\personal\\data\" \"%V\""
```

**To apply:**
1. Save the file with `.reg` extension
2. Double-click the file
3. Click **Yes** when prompted
4. Refresh File Explorer (F5)

### Method 2: Using PowerShell Commands

Run **PowerShell as Administrator**:

```powershell
# Create HKCR drive
New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT

# Work Profile - Folder
New-Item -Path "HKCR:\Directory\shell\VSCodeWork" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodeWork" -Name "(Default)" -Value "Open with VS Code - Work"
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodeWork" -Name "Icon" -Value "C:\vscode\vscode.ico"
New-Item -Path "HKCR:\Directory\shell\VSCodeWork\command" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodeWork\command" -Name "(Default)" -Value '"C:\Program Files\Microsoft VS Code\Code.exe" --user-data-dir "C:\vscode\work\data" "%1"'

# Personal Profile - Folder
New-Item -Path "HKCR:\Directory\shell\VSCodePersonal" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodePersonal" -Name "(Default)" -Value "Open with VS Code - Personal"
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodePersonal" -Name "Icon" -Value "C:\vscode\vscode-alt.ico"
New-Item -Path "HKCR:\Directory\shell\VSCodePersonal\command" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodePersonal\command" -Name "(Default)" -Value '"C:\Program Files\Microsoft VS Code\Code.exe" --user-data-dir "C:\vscode\personal\data" "%1"'

# Work Profile - Background
New-Item -Path "HKCR:\Directory\Background\shell\VSCodeWork" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodeWork" -Name "(Default)" -Value "Open with VS Code - Work"
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodeWork" -Name "Icon" -Value "C:\vscode\vscode.ico"
New-Item -Path "HKCR:\Directory\Background\shell\VSCodeWork\command" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodeWork\command" -Name "(Default)" -Value '"C:\Program Files\Microsoft VS Code\Code.exe" --user-data-dir "C:\vscode\work\data" "%V"'

# Personal Profile - Background
New-Item -Path "HKCR:\Directory\Background\shell\VSCodePersonal" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodePersonal" -Name "(Default)" -Value "Open with VS Code - Personal"
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodePersonal" -Name "Icon" -Value "C:\vscode\vscode-alt.ico"
New-Item -Path "HKCR:\Directory\Background\shell\VSCodePersonal\command" -Force | Out-Null
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodePersonal\command" -Name "(Default)" -Value '"C:\Program Files\Microsoft VS Code\Code.exe" --user-data-dir "C:\vscode\personal\data" "%V"'
```

### Updating Icons After Setup

If you want to change icons later:

**PowerShell (as Administrator):**
```powershell
New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT

Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodeWork" -Name "Icon" -Value "C:\vscode\vscode.ico"
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodeWork" -Name "Icon" -Value "C:\vscode\vscode.ico"
Set-ItemProperty -Path "HKCR:\Directory\shell\VSCodePersonal" -Name "Icon" -Value "C:\vscode\vscode-alt.ico"
Set-ItemProperty -Path "HKCR:\Directory\Background\shell\VSCodePersonal" -Name "Icon" -Value "C:\vscode\vscode-alt.ico"
```

---

## Testing Your Setup

### 1. Test Desktop Shortcuts
- Double-click **VS Code - Work** shortcut
- VS Code should open
- Sign in with your work GitHub account
- Close VS Code
- Double-click **VS Code - Personal** shortcut
- Sign in with your personal GitHub account

### 2. Test Context Menu
- Right-click any folder
- You should see:
  - Open with VS Code - Work
  - Open with VS Code - Personal
- Click one to test

### 3. Verify Separation
- Open both profiles simultaneously
- Check GitHub account in each (click account icon, bottom-left)
- They should show different accounts

### 4. Verify Profile Paths

In each VS Code window:
- Go to **Help** → **About**
- Check "User data directory"
- Should show `C:\vscode\work\data` or `C:\vscode\personal\data`

---

## Troubleshooting

### Context Menu Not Appearing

**Solution 1: Refresh File Explorer**
- Press `F5` in File Explorer
- Or close and reopen File Explorer

**Solution 2: Verify Registry Entries**

PowerShell:
```powershell
New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT
Test-Path "HKCR:\Directory\shell\VSCodeWork"
Test-Path "HKCR:\Directory\shell\VSCodePersonal"
```

Both should return `True`.

**Solution 3: Check VS Code Installation Path**

If VS Code is installed elsewhere, update the path:
```powershell
Get-Command code | Select-Object Source
```

Then update the registry entries with the correct path.

### Icons Not Showing

**Check:**
1. Icon files exist at specified paths
2. Files have `.ico` extension
3. File paths in registry match actual locations
4. Refresh File Explorer (F5)

### "Access Denied" Error

**Solution:**
- Run PowerShell as **Administrator**
- Win+X → Terminal (Admin)

### GitHub Account Not Staying Logged In

**Solution:**
1. Sign out from both profiles
2. Open Work profile → Sign in with work account
3. Close completely
4. Open Personal profile → Sign in with personal account
5. Each profile stores credentials independently

### Original "Open with Code" Still Uses Old Account

**This is normal!**
- The default context menu still uses `C:\Users\YOUR_USERNAME\AppData\Roaming\Code`
- Use your new custom menu entries instead
- Or sign into the original with a different account

---

## Removing the Setup

### Remove Context Menu Entries

**Create file: `remove-vscode-context-menu.reg`**

```reg
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\Directory\shell\VSCodeWork]
[-HKEY_CLASSES_ROOT\Directory\shell\VSCodePersonal]
[-HKEY_CLASSES_ROOT\Directory\Background\shell\VSCodeWork]
[-HKEY_CLASSES_ROOT\Directory\Background\shell\VSCodePersonal]
```

Double-click to remove entries.

**Or use PowerShell (as Admin):**
```powershell
New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT
Remove-Item -Path "HKCR:\Directory\shell\VSCodeWork" -Recurse -Force
Remove-Item -Path "HKCR:\Directory\shell\VSCodePersonal" -Recurse -Force
Remove-Item -Path "HKCR:\Directory\Background\shell\VSCodeWork" -Recurse -Force
Remove-Item -Path "HKCR:\Directory\Background\shell\VSCodePersonal" -Recurse -Force
```

### Remove Profile Directories

**PowerShell:**
```powershell
Remove-Item -Path "C:\vscode" -Recurse -Force
```

### Remove Desktop Shortcuts

Simply delete the shortcuts from your desktop.

---

## File Structure Summary

```
C:\vscode\
├── work\
│   └── data\           (Work profile data)
├── personal\
│   └── data\           (Personal profile data)
├── vscode.ico          (Work icon - optional)
├── vscode-alt.ico      (Personal icon - optional)
├── vscode-work.bat     (Work launcher)
└── vscode-personal.bat (Personal launcher)
```

---

## Quick Reference Commands

### Verify Setup
```powershell
# Check if profiles exist
Test-Path "C:\vscode\work\data"
Test-Path "C:\vscode\personal\data"

# Check registry entries
New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT
Test-Path "HKCR:\Directory\shell\VSCodeWork"
```

### Launch from Command Line
```powershell
# Work profile
C:\vscode\vscode-work.bat "C:\Projects\WorkProject"

# Personal profile
C:\vscode\vscode-personal.bat "C:\Projects\PersonalProject"
```

---

## Tips & Best Practices

1. **Keep profiles clean**: Only install extensions you need in each profile
2. **Use descriptive names**: Consider renaming profiles based on your workflow
3. **Backup configurations**: Copy `C:\vscode\` folder periodically
4. **SSH Keys**: Consider setting up separate SSH keys for each GitHub account
5. **Regular updates**: Update VS Code through either profile - it's the same installation

---

## Additional Resources

- [VS Code Profiles Documentation](https://code.visualstudio.com/docs/configure/profiles)
- [Managing Multiple GitHub Accounts with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [VS Code Command Line Interface](https://code.visualstudio.com/docs/editor/command-line)

---

## Version History

- **v1.0** - Initial setup guide
- Created: January 2026

---

**Note:** This guide assumes VS Code is installed at `C:\Program Files\Microsoft VS Code\Code.exe`. If your installation is elsewhere, adjust paths accordingly.