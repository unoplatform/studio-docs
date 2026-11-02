---
uid: Uno.PlatformStudio.WindowsInstall
---

# Installing and Updating on Windows

You install Uno Platform Studio App on Windows from the download page. Windows installs it and keeps it up to date, so the app has no updater of its own. This page explains what to expect, and how to check an installation when something looks wrong.

In the Start menu and in **Settings > Apps**, the app appears as **Uno Studio**, published by **Uno Platform**.

## Requirements

- Windows 10 version 2004 or later, or Windows 11.
- A 64-bit Intel, AMD or Arm PC. The same download serves all of them, and each PC gets the build made for its processor.
- No administrator rights, and no other software to install first.

Windows refuses to install the app on an older version of Windows.

## Installing

1. Open the Uno Platform Studio App download page and choose **Windows**.
2. Open the file you downloaded. Windows' App Installer opens and names **Uno Platform** as the publisher.
3. Choose **Install**.
4. Start **Uno Studio** from the Start menu, and sign in through your browser.

> [!NOTE]
> The download is a small installer file, not the app itself. It tells Windows where updates come from, so only an installation made from the download page receives updates.

## How Updates Work

Updates need no action from you, and you are never prompted.

1. Each time you start the app, Windows checks for a newer version.
2. If there is one, Windows downloads it in the background. Your session is not interrupted.
3. The **next** time you start the app, Windows installs the downloaded version first, then starts it.

Closing the app does not install the update; the next start does. Windows also checks in the background about every eight hours, so an installation you rarely start still receives updates.

Your sign-in, settings and window position carry over to the new version. Your projects are not touched.

## Expected `0x80073D02` Entries in the Windows Log

Windows tries to install an update as soon as it has downloaded it. While the app is running, that attempt fails with error `0x80073D02`, because the app is in use. Windows keeps the downloaded version and installs it at the next start.

You will therefore see one `0x80073D02` entry for each update in Windows' deployment log, `Microsoft-Windows-AppXDeploymentServer/Operational`. **These entries are expected and need no action.**

To read the log from PowerShell:

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-AppXDeploymentServer/Operational' -MaxEvents 50
```

## Checking an Installation's Update Settings

If an installation stops receiving updates, check that it still follows the download feed. In PowerShell:

```powershell
$package = Get-AppxPackage -Name UnoPlatform.UnoStudio
$package.Version
Get-AppPackageAutoUpdateSettings -PackageFamilyName $package.PackageFamilyName
```

A healthy installation shows:

| Setting | Expected value |
| ------- | -------------- |
| `AppInstallerUri` | The address of the Windows download |
| `CheckForUpdatesOnLaunch` | `True` |
| `AutomaticBackgroundTaskUpdatesEnabled` | `True` |
| `HoursBetweenUpdateChecks` | `0` |

If the command shows no `AppInstallerUri`, the app was not installed from the download page, and it will never update. Uninstall it, then install it again from the download page.

## Windows LTSC and Windows Server

The LTSC and Server editions of Windows do not include App Installer, so Windows cannot open the downloaded file. Install from PowerShell instead:

1. On the download page, right-click the **Windows** download and copy its address.
2. Run:

   ```powershell
   Add-AppxPackage -AppInstallerFile <the address you copied>
   ```

An app installed this way receives updates in the same way as above.

> [!IMPORTANT]
> Always pass the download address with `-AppInstallerFile`. Installing the app package file directly works, but that installation never receives updates.

## Uninstalling

Open **Settings > Apps > Installed apps**, find **Uno Studio**, and choose **Uninstall**. You can also right-click **Uno Studio** in the Start menu and choose **Uninstall**.

No administrator rights are needed. Projects you saved to a folder of your choice are not removed.
