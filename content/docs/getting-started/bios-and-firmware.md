---
title: BIOS/UEFI and Firmware updates
---

# BIOS/UEFI and other firmware updates

LSUClient will install BIOS/UEFI updates silently when possible.

Like with any other package, those for which a silent installation is supported will have their
`.Installer.Unattended` property set to `$True`. If you get a BIOS/UEFI update for which
`.Installer.Unattended` is `$False`, that means it uses a type of installer/flasher for which
either no silent install options exist or for which LSUClient doesn't support performing silent installs yet.

When you install such a non-silent package its setup routine will be invoked with the default arguments in the same way System Update would.  
Likely this will mean a graphical wizard will come up and wait for you to confirm or cancel the update.

## Handling reboots

It is important to know that some Lenovo computers require a reboot to apply BIOS updates
while other models require a shutdown - the BIOS will then wake the machine from the power-off state,
apply the update and boot back into Windows. Other, non-BIOS firmware updates typically always require a reboot.
So as to not interrupt a deployment or someone working, this module will never initiate reboots
or shutdowns on its own, however it's easy for you to:

1. Capture the `PackageInstallResult` objects returned by `Install-LSUpdate`, e.g.:

    ```powershell
    [array]$results = Install-LSUpdate -Package $updates
    ```

2. Then test for the `PendingAction` values `REBOOT_MANDATORY` or `SHUTDOWN` and handle them in your script:
    ```powershell
    if ($results.PendingAction -contains 'REBOOT_MANDATORY') {
        # reboot immediately or set a marker for yourself to perform the reboot shortly
    }
    if ($results.PendingAction -contains 'SHUTDOWN') {
        # shutdown immediately or set a marker for yourself to perform the shutdown shortly
    }
    ```
    If you prefer to loop through the updates one-by-one and handle their result immediately, you can use `-eq` or a switch statement:
    ```powershell
    foreach ($update in $updates) {
        $result = Install-LSUpdate -Package $update
        switch ($result.PendingAction) {
            # your logic here
        }
    }
    ```

## SaveBIOSUpdateInfoToRegistry Parameter

{{< hint danger >}}
This parameter is deprecated and only documented for the sake of helping move existing scripts away from it.

It may be removed in a future major release of LSUClient. Use the technique described [above]({{< relref "#handling-reboots" >}}) instead.
{{< /hint >}}

There is also a `-SaveBIOSUpdateInfoToRegistry` parameter on `Install-LSUpdate`.

Prior to version 1.4.0, this used to be the only way for LSUClient to communicate a required power cycle back to you. 

When `Install-LSUpdate` is called with this parameter and it successfully installs a BIOS update,
it will write some registry keys to `HKLM\Software\LSUClient\BIOSUpdate`, including the string
`ActionNeeded` which will contain either `"REBOOT"` or `"SHUTDOWN"`.

However, using the `-SaveBIOSUpdateInfoToRegistry` parameter is no longer recommended because,
as the name implies, it only sets those registry keys when installing BIOS/UEFI updates and not
for any other kinds of firmware updates that might require a reboot just the same.

To summarize the differences and aid the transition, the following table compares the values
to expect in different scenarios:

  | Scenario                 | PendingAction property | "ActionNeeded" registry value set by `-SaveBIOSUpdateInfoToRegistry` |
  |:-------------------------|:-----------------------|-------------------|
  | ThinkPad BIOS update     | REBOOT_MANDATORY       | REBOOT            |
  | ThinkCentre BIOS update  | SHUTDOWN               | SHUTDOWN          |
  | Any Reboot Type 5 update | REBOOT_MANDATORY       |                   |
  | Any Reboot Type 3 update | REBOOT_SUGGESTED       |                   |
  | Any Reboot Type 0 update | NONE                   |                   |
  | Any unsuccessful update  | NONE                   |                   |

As you can see, the `PendingAction` property is always set and more explicit and granular in communicating whether a power cycle is needed or not.

## Excluding BIOS and/or firmware updates

If you want to simply not install any BIOS/UEFI updates, I recommend filtering them by `Type` and possibly `Category` and `Title` as a fallback.

{{< hint warning >}}
Not all packages have type information, sometimes `Type` is `$null` so don't rely on this property alone
{{< /hint >}}

{{< hint warning >}}
Packages sourced from internal repositories created with "Lenovo Update Retriever" never have `Category` information.

In that scenario it is best to either not include BIOS updates in your repository at all or to filter them by their IDs before installing.
{{< /hint >}}

Filtering out BIOS/UEFI updates:

```powershell
$updates = Get-LSUpdate |
    Where-Object { $_.Type -ne 'BIOS' } |
    Where-Object { $_.Category -notmatch "BIOS|UEFI" } |
    Where-Object { $_.Title -notmatch "BIOS|UEFI" }
```
Filtering out other firmware updates:
```powershell
$updates = Get-LSUpdate |
    Where-Object { $_.Type -ne 'Firmware' } |
    Where-Object { $_.RebootType -ne 5 } |
    Where-Object { $_.Category -notlike "*Firmware*" } |
    Where-Object { $_.Title -notlike "*Firmware*" }
```

## Recommended reading

https://support.lenovo.com/de/en/solutions/ht507859-bios-flashing-sccm-support-thinkcentre-thinkstation  
https://thinkdeploy.blogspot.com/2019/06/what-are-reboot-delayed-updates.html
