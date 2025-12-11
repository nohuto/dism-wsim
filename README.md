# DISM-WSIM - Not Finished!
Guide on how to create your own customized and automated Windows installation.


## Preperation

Installing WSIM:
```ps
winget install Microsoft.WindowsADK
```

Downloading the preferred image:
- [massgrave.dev](https://massgrave.dev/genuine-installation-media)
- [uupdump.net](https://uupdump.net/)

## Installation / Configuration Folder
Creating your personal `Installation`/`Configuration` or whatever you want to call it folder, example structure:
```mathematica
C:\
└── Installation
    │
    ├── !Notes.txt
    ├── 00 - Config.bat
    ├── 01 - Scoop.ps1
    ├── 02 - Winget.ps1
    ├── 03 - Driver.ps1
    │
    ├── Folders
    │   ├── Config
    │   │   ├── !Tools
    │   │   ├── 00 - Toggle
    │   │   ├── 01 - Components
    │   │   ├── 02 - Services
    │   │   ├── 03 - Power
    │   │   ├── 04 - Peripheral
    │   │   ├── 05 - Network
    │   │   ├── 06 - GPU
    │   │   ├── 07 - Apps
    │   │   ├── Final.bat
    │   │   ├── Privacy.bat
    │   │   ├── System.bat
    │   │   └── Visibility.bat
    │   │
    │   └── Image
    │
    └── Settings
```
```ps
# Scoop.ps1

cscript //nologo "$env:windir\system32\slmgr.vbs" /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
cscript //nologo "$env:windir\system32\slmgr.vbs" /ato

cmd /c start /wait "C:\Installation\Settings\vcredist64.exe" /passive /norestart
cmd /c start /wait "C:\Installation\Settings\vcredist86.exe" /passive /norestart

iwr get.scoop.sh -OutFile "$env:temp\Scoop.ps1"
powershell -File "$env:temp\Scoop.ps1" -RunAsAdmin -Wait
```
```ps
# Winget.ps1 - Software Installation Examples
scoop install winget

winget install 7zip.7zip --accept-package-agreements --accept-source-agreements
winget install StartIsBack.StartAllBack --scope machine
winget install WinsiderSS.SystemInformer
winget install Notepad++.Notepad++ --override "/"
winget install Alex313031.Thorium.AVX2
winget install Valve.Steam
winget install Equicord.Equibop
winget install voidtools.Everything.Alpha
winget install Skillbrains.Lightshot
winget install Guru3D.Afterburner.Beta
winget install OBSProject.OBSStudio
winget install qBittorrent.qBittorrent
winget install Microsoft.WindowsADK
winget install Microsoft.VisualStudioCode
winget install Microsoft.WindowsTerminal
winget install Klocman.BulkCrapUninstaller
Invoke-Expression "& { $(Invoke-WebRequest -UseBasicParsing 'https://raw.githubusercontent.com/SpotX-Official/spotx-official.github.io/main/run.ps1') } -v 1.2.13.661.ga588f749-4064 -confirm_spoti_recomended_over -block_update_on -podcasts_off -adsections_off"

$dir = "$env:localappdata\Noverse"
$dest = "$env:localappdata\Noverse\NVFetch.ps1"
if (!(Test-Path $dir)) { New-Item -ItemType Directory -Path $dir -Force | Out-Null }
Copy-Item "C:\Installation\Settings\NVFetch.ps1" -Destination $dest -Force
if (!(Test-Path $profile)) { New-Item -ItemType File -Path $profile -Force | Out-Null }
Add-Content $profile -Value "`nfunction nvfetch { & `"$dest`" @args }"
. $profile

$destblur = "C:\Program Files\ExplorerBlur"
$zip  = "$env:temp\ExplorerBlur.zip"
Invoke-WebRequest "https://github.com/Maplespe/ExplorerBlurMica/releases/download/2.0.1/Release_x64.zip" -OutFile $zip
if (!(Test-Path $destblur)) { New-Item -ItemType Directory -Path $destblur -Force | Out-Null }
& 7z x $zip "-o$destblur" -y
Start-Process "cmd.exe" "/c `"$destblur\Release\register.cmd`"" -Verb RunAs -Wait
```

## Personalized `autounattend.xml` Creation

Creating `autounattend.xml` using WSIM:
1. `windowsPE` phase
```mathematica
Microsoft-Windows-International-Core-WinPE
  Input Locale - 0409:00000407
  SystemLocale - en-US
  UILanguage - en-US
  UILanguageFallback - en-US
  UserLocale - en-US

  SetupUILanguage
    UILanguage - en-US
```


## Modifying the ISO

Extracting the `.iso`:
```ps
& 7z x "$home\Desktop\Stock.iso" -o"$home\Desktop\Stock" -y
```
Removing indexes (only leaving `Windows 11 IoT Enterprise LTSC`:
```ps
$wimpath = "$home\Desktop\Stock\sources\install.wim"
# Examples
$remindex = @(
    #24H2
    "Windows 11 Enterprise LTSC",
    "Windows 11 IoT Enterprise Subscription LTSC",
    #25H2
    "Windows 11 Enterprise",
    "Windows 11 IoT Enterprise Subscription"
)

$images = Get-WindowsImage -ImagePath $wimpath
foreach ($name in $remindex) {
    if ($images.name -contains $name) {
        Remove-WindowsImage -ImagePath $wimpath -Name $name
    }
}
```
Creating the mount folder (`Index 1 = Windows 11 IoT Enterprise LTSC`):
```ps
$mountpath = "$home\Desktop\Mount"
if (!(Test-Path $mountpath)) { New-Item -ItemType Directory -Path $mountpath -Force | Out-Null}
Mount-WindowsImage -ImagePath "$home\Desktop\Stock\sources\install.wim" -Index 1 -Path "$home\Desktop\Mount"
```
Removing Edge (optional):
```ps
Remove-Item "$mountpath\Program Files (x86)\Microsoft\Edge" -Recurse -Force
Remove-Item "$mountpath\Program Files (x86)\Microsoft\EdgeCore" -Recurse -Force
Remove-Item "$mountpath\Program Files (x86)\Microsoft\EdgeUpdate" -Recurse -Force
```

See the currently installed/enabled components:
```ps
Get-WindowsPackage -Path $mountpath
Get-WindowsOptionalFeature -Path $mountpath
Get-WindowsCapability -Path $mountpath
```
Removing packages:
```ps
# Examples
$packages = @(
  "Microsoft-Windows-Hello-Face-Package*",
  "Microsoft-Windows-MSPaint-FoD-Package*",
  "Microsoft-Windows-Notepad-FoD-Package*",
  "Microsoft-Windows-PowerShell-ISE-FOD-Package*",
  "Microsoft-Windows-SnippingTool-FoD-Package*",
  "Microsoft-Windows-StepsRecorder-Package*",
  "Microsoft-Windows-TabletPCMath-Package*",
  "Microsoft-Windows-Wallpaper-Content-Extended-FoD-Package*",
)

foreach ($p in $packages) { Get-WindowsPackage -Path $mountpath | ? { $_.PackageName -like $p -and $_.State -eq 'Installed' } | % { Remove-WindowsPackage -Path $mountpath -PackageName $_.PackageName } }
```
Disabling optional features:
```ps
# Examples
$featuredisable = @(
    "MicrosoftWindowsPowerShellV2Root",
    "MicrosoftWindowsPowerShellV2",
    "WorkFolders-Client",
    "SmbDirect",
    "Printing-PrintToPDFServices-Features",
    "Recall",
    "Microsoft-RemoteDesktopConnection",
    "Printing-Foundation-Features",
    "Printing-Foundation-InternetPrinting-Client",
)

foreach ($f in $featuredisable) { Disable-WindowsOptionalFeature -Path $mountpath -FeatureName $f }

$featureenable = @(
    #"LegacyComponents",
    "DirectPlay"
)

foreach ($f in $featureenable) { Enable-WindowsOptionalFeature -Path $mountpath -FeatureName $f -All }
```
Removing capabilities:
```ps
# Examples
$capability = @(
    "App.StepsRecorder*",
    "Browser.InternetExplorer*",
    "Hello.Face*",
    "MathRecognizer*",
    "Microsoft.Wallpapers.Extended*",
    "Microsoft.Windows.MSPaint*",
    "Microsoft.Windows.Notepad*",
    "Microsoft.Windows.PowerShell.ISE*",
    "Microsoft.Windows.SnippingTool*",
    "Microsoft.Windows.Wifi.Client*",
    "OneCoreUAP.OneSync*"
)

foreach ($c in $capability) { Get-WindowsCapability -Path $mountpath | ? { $_.Name -like $c } | % { Remove-WindowsCapability -Path $mountpath -Name $_.Name } }
```
Adding drivers:
```ps
# '$home\Desktop\Driver' should include the driver files
Add-WindowsDriver -Path "$home\Desktop\Mount" -Driver "$home\Desktop\Driver" -Recurse -ForceUnsigned
```
Moving your post install folder & `autounattend.xml`:
```ps
Copy-Item "C:\Installation" -Destination $mountpath -Recurse -Force
Copy-Item "C:\Installation\Folders\Image\autounattend.xml" -Destination "$home\Desktop\Stock" -Force
```
Saving the changes & dismounting the image:
```ps
Dismount-WindowsImage -Path "$home\Desktop\Mount" -Save
```
```ps
& "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools\amd64\Oscdimg\oscdimg.exe" -m -o -u2 -udfver102 -l"Enterprise" -bootdata:1#pEF,e,b"$home\Desktop\Stock\efi\microsoft\boot\efisys.bin" "$home\Desktop\Stock" "$home\Desktop\Enterprise.iso"
```


> https://github.com/nohuto/windows-powershell-docs/blob/main/docset/winserver2025-ps/Dism/Dism.md
> https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/oscdimg-command-line-options?view=windows-11
