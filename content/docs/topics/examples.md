---
title: Basic Examples
weight: 10
bookToc: false
---

# Basic Examples

These are some examples to try out.
For all available parameters and guidance on how to use them run `Get-Help -Detailed` on the functions in this module.

### Get available updates
```powershell
Get-LSUpdate
```

### Get and install available updates
```powershell
$updates = Get-LSUpdate
$updates | Install-LSUpdate -Verbose
```

### Get and install available updates, with some simple progress output
```powershell
$updates = Get-LSUpdate
$i = 1
foreach ($update in $updates) {
    Write-Host "Installing update $i of $($updates.Count): $($update.Title)"
    Install-LSUpdate -Package $update -Verbose
    $i++
}
```

### Install only packages that can be installed silently and non-interactively
```powershell
$updates = Get-LSUpdate | Where-Object { $_.Installer.Unattended }
$updates | Save-LSUpdate -Verbose
$updates | Install-LSUpdate -Verbose
```

Filtering out non-unattended packages like this is strongly recommended when using this module in MDT, SCCM, PDQ,
remote execution via PowerShell Remoting, ssh or any other situation in which you run these commands remotely
or as part of an automated process. Packages with installers that are not unattended may force reboots or
attempt to start a GUI setup on the machine and, if successful, halt until someone clicks through the dialogs.

### Get all available packages
```powershell
$updates = Get-LSUpdate -All
```
By default, `Get-LSUpdate` only returns "needed" updates. Needed updates are those that are applicable to
the system and not yet installed. If you want to retrieve all available packages instead, use `Get-LSUpdate -All`.
To filter out unneeded packages later, just look at the `IsApplicable` and `IsInstalled` properties.
The default logic is equivalent to:
`Get-LSUpdate -All | Where-Object { $_.IsApplicable -and -not $_.IsInstalled }`

### Download drivers for another computer
```powershell
Get-LSUpdate -Model '20LS' -All | Save-LSUpdate -Path 'C:\Drivers\20LS' -ShowProgress
```
Using the `-Model` parameter of `Get-LSUpdate` you can retrieve packages for another computer model.
In this case you almost always want to use `-All` too so that the packages found are not filtered against your computer and all packages are downloaded.

Ever since LSUClient Version 1.3.3 you can also pass full-length MTM/model names such as `20K70000GE` to this parameter.

