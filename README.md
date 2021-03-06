# Introduction 
Automatically apply settings referenced in white papers:</br>
"Optimizing Windows 10, version 1909, for a Virtual Desktop Infrastructure (VDI) role"  
URL: https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/rds_vdi-recommendations-1909  
This information will be updated once later versions are published on docs.microsoft.com.  
A new version of this paper for Windows 10 2004 is pending publication as of 06/18/2020.

# Getting Started

 ## REFERENCES:
 https://social.technet.microsoft.com/wiki/contents/articles/7703.powershell-running-executables.aspx  
 https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item?view=powershell-6  
 https://blogs.technet.microsoft.com/secguide/2016/01/21/lgpo-exe-local-group-policy-object-utility-v1-0/  
 https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-service?view=powershell-6  
 https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item?view=powershell-6  
 https://msdn.microsoft.com/en-us/library/cc422938.aspx

 ## CUSTOMIZATION
 You can customize your deployments by editing the .JSON or .TXT input files.</br>
 .TXT input file (DefaultUserSettings.txt): If you wish to change registry settings set by this optimization package, you can edit this file and make those changes.
 .JSON input files: .JSON files have a code block for each respective setting. You can edit the .JSON input files and remove a particular block.  For example:</br>
 If you wish to keep the Calculator UWP app, locate and delete this block of code from the file 'AppxPackages.Json':</br>
```
{
  "AppxPackage": "Microsoft.WindowsCalculator",
  "VDIState": "Disabled",
  "URL": "https://www.microsoft.com/en-us/p/windows-calculator/9wzdncrfhvn5",
  "Description": "Microsoft Calculator app"
},
```

 ## DEPENDENCIES
 1. LGPO.EXE (available at https://www.microsoft.com/en-us/download/details.aspx?id=55319) stored in the 'LGPO' folder.
 2. Download and expand the entire repository to your target machine.

**NOTE:** This script now takes just a few minutes to complete on the reference (gold) device. The total runtime will be presented at the end, in the status output messages.  
A prompt to reboot will appear when the script has comoletely finished running. Wait for this prompt to confirm the script has successfully completed.  
Also, the "-verbose" parameter in PowerShell directs the script to provide descriptive output as the script is running.

 ## Full Instructions (for Windows 10 2004, OR Windows 10 1909)
 **NOTE** The PowerShell command to start the optimization tool is the same on 1909 or 2004
1. Download to the reference machine, in a folder (ex. C:\Optimize), this entire GitHub repository:</br>
2. Start PowerShell elevated
3. In PowerShell, change directory to the scripts folder (ex. C:\Optimize)
4. Run the following PowerShell commands:</br></br>
        ``Set-ExecutionPolicy -ExecutionPolicy RemoteSigned``</br>
        ``.\Win10_VirtualDesktop_Optimize.ps1 -WindowsVersion <Windows Version> -Verbose``</br></br>
5. When complete, you should see a prompt to restart.  You do not have to restart right away.

# IMPORTANT ISSUE (06/18/2020) (Resolved)
Windows 10 Multi-Session has different 'Privilege Rights' than standard Windows 10.  To this point, these optimization scripts have been based on "normal" Windows 10.</br>
As of 6/18/2020, the "Secedit" and "Audit" files have been deleted from the LGPO backup. Now the only settings applied are local registry settings</br>
Therefore, any differences in security settings based on SKU will not be change by this optimization tool.</br>

# Note on disk cleanup (06/11/2020)
Starting with the 2004 version of these scripts, we no longer invoke the Disk Cleanup Wizard (Cleanmgr.exe).  DCW is near end-of-life, but also sometimes "hangs" during running of the scripts.  Instead some basic disk cleanup has been incorporated into the 'Win10_VirtualDesktop_Optimize.ps1' script.  There are logs, traces, and event log files deleted.  If you wish to maintain log files, you can edit the .PS1 script and remove those entries.

# Note on Servicing (06/11/2020)
The 2004 scripts, as currently configured, pause all updates, including Quality Updates.  These settings do not affect Windows Defender, which gets it updates independently. If you want to allow your target machine(s) to contact Windows Update to download and apply updates, you can change the following group policy setting either locally, or in central group policy:

`Computer Configuration\Administrative Templates\Windows Components\Windows Update\Windows Update for Business\`

`Select when Quality Updates are received	Not configured`

You would also want to reset the 'Update Orchestrator' service to it's initial setting of "Automatic (Delayed Start)".

# MINOR ISSUE (06/11/2020)
We had removed the "OneConnect" (Mobile Plans) entry from the input file 'AppxPackages.json', because that UWP app is no longer in Windows 10, starting with 2004.  However, the 2004 scripts are backward compatible with 1909, though have not been tested on any build prior to 1909.  Therefore the 'OneConnect' app entry was added back to the AppxPackages.json file.

# 1909 Medium-impact ISSUE (05/11/2020) (Resolved)
**WINDOWS UPDATE NOT WORKING**
With the settings included in the LGPO backup, which is restored to the target during the processing of these scripts, if you attempt to run Windows Update manually, you may not be able to connect.  This is because Feature Updates are disabled via local policy in these scripts.  If you set all Windows Update policies back to "not configured", then run "GPUPDATE /force", now your machine will connect to Windows Update.
The reason these settings are in place in these scripts, is in case you deploy these to a target that is Internet connected, your VM may try to "Feature Update" to the current Windows 10 build, which is termed "2004" (as of May 11, 2020).  The settings in place currently, prevent Feature Updates, but also seem to inhibit just downloading monthly updates to the current build.
To address this for implementations that prefer to allow Windows Update, a new "fork" of these optimization scripts has been created under the main code folder.  The new folder is called "1909_WindowsUpdateEndabled".  Within this folder, the local policy settings (LGPO) have all Windows Update settings "not configured".
If you need to have Windows Update enabled out of the gate, try the scripts under this folder and raise an issue if any problems are found.

# NOTE (05/11/2020): New settings added to default user profile
Disable "Inking & typing personalization" in Settings

# 1909 Low-impact ISSUE (04/29/2020) (Resolved)
**Apps running in the background**
Several of the built-in UWP apps, such as Skype, Phone, and Photos, will start processes and run in the background, even though the user has not started the app(s).  On a single machine this is near-zero impact, but on multi-session Windows, it can be a slightly larger impact issue.  There is a setting in the 'Settings' app, under 'Background apps' that allows you to control this behavior on a per-user basis.  However, there is currently no way to change this behavior as a global setting, other than to completely uninstall the app.

If you would like to keep one or more of these apps in your image, and still control the background behavior, you can edit the default user registry hive and set the following settings:

"HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.Windows.Photos_8wekyb3d8bbwe /v Disabled /t REG_DWORD /d 1 /f
"HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.Windows.Photos_8wekyb3d8bbwe /v DisabledByUser /t REG_DWORD /d 1 /f
"HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.SkypeApp_kzf8qxf38zg5c /v Disabled /t REG_DWORD /d 1 /f
"HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.SkypeApp_kzf8qxf38zg5c /v DisabledByUser /t REG_DWORD /d 1 /f
"HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.YourPhone_8wekyb3d8bbwe /v Disabled /t REG_DWORD /d 1 /f
"HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.YourPhone_8wekyb3d8bbwe /v DisabledByUser /t REG_DWORD /d 1 /f

You could also set these settings with Group Policy Preferences, and should take effect after a log off and log back on.

# Low-impact ISSUE (04/22/2020) (Resolved)

In some virtual environments, such as Azure Windows Virtual Desktop, some of the application windows will have no border.  An example is Windows File Explorer.  You can replicate this by opening Wordpad and File Explorer, then move then around and note that you may not see a border where one app starts and the other ends.
One of the optimizations in the latest drop changes the Visual Effects settings (found in System Properties) to reduce animations and effects, while still maintaining a good user experience such as "smoothing screen fonts".
The other two optimizations: "show shadows under mouse pointer" and "Show shadows under windows" will enable a shadow effect around the windows like File Explorer, so that the border of the app is now visible.
These settings are written to the default user profile registry hive, so would apply only to users whose profile is created after these optimizations run, and on this computer.

# Low-impact ISSUE (04/20/2020) (Resolved)
Previously these scripts had a local policy setting at this location set to disabled:

**Local Computer Policy \ Computer Configuration \ Administrative Templates \ System \ Internet Communication Management \ Internet Communication settings**
```
Turn off Windows Network Connectivity Status Indicator active tests
```
With the active tests disabled, Office 365 is not able to contact it's licensing service, and therefore would not run any of the Office apps.  This setting has been changed back to "Not configured" in the included LGPO file.

# IMPORTANT ISSUE (01/17/2020) (Resolved)
IMPORTANT: There is a setting in the current LGPO files that should not be set by default. As of 1/17/10...
a fix has been checked in to the "Pending" branch.  Once we confirm that resolves the issue we will merge...
into the "Master" branch.  The issue is that Windows will not check certificate information, and thus...
program installations could fail.  The temporary workaround is to open GPEDIT.MSC on the reference image...
The set the policy to "not configured".  Here is the location of the policy setting:

**Local Computer Policy \ Computer Configuration \ Administrative Templates \ System \ Internet Communication Management \ Internet Communication settings**

```
Turn off Automatic Root Certificates Update
```
