---
weight: 20
---

# Best Practices

## 1. Search for and install updates a few times in a loop

Some packages and updates depend on others already being installed, so they might not show up during the first search for updates.
If you want to make sure "everything" has been installed and/or updated it is best to re-run 
`Get-LSUpdate` and `Install-LSUpdate` a few times or until no more updates are found.

{{< details title="Example" open=false >}}

This is a minimal example implementation of this best practice for demonstration purposes:

```powershell {linenos=table}
$MaxRounds = 3
for ($Round = 1; $Round -le $MaxRounds; $Round++) {
    Write-Host "Starting round $Round"
    $updates = Get-LSUpdate
    Write-Host "$($updates.Count) updates found"

    if ($updates.Count -eq 0) {
        break;
    }

    $updates | Install-LSUpdate -Verbose
}
```

{{< /details >}}

## 2. Download all packages before starting to install any

Installing a network (NIC, WiFi adapter) driver can cause a short loss of network connectivity. When you feed packages found by
`Get-LSUpdate` directly to `Install-LSUpdate` they will each be individually downloaded and then installed, therefore a network
driver installation can cause the following package downloads to fail (see [GitHub issue #41](https://github.com/jantari/LSUClient/issues/41)).
Downloading all packages you intend to install beforehand with `Save-LSUpdate` will prevent this potential problem.

{{< details title="Example" open=false >}}

This is a minimal example implementation of this best practice for demonstration purposes:

```powershell {linenos=table}
# Find updates
$updates = Get-LSUpdate

# Download them all to the local disk
$updates | Save-LSUpdate

# Then install
$updates | Install-LSUpdate
```

{{< /details >}}

{{< hint info >}}
If you download packages to a custom location with `Save-LSUpdate -Path` you have to pass that same path to `Install-LSUpdate -Path`
in order for it to find the previously downloaded files. If `Install-LSUpdate` cannot find the expected files locally, it will still
attempt to fetch them from the repository.
{{< /hint >}}

## 3. Write a logfile

Especially when running unattended or remotely, it is crucial to have a good logfile that details what happened and allows you to find
the cause of a problem in retrospect.

If you don't know where to start, start with [`Start-Transcript`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript?view=powershell-5.1) to log whatever is being output to the console.
If you would like more insight into what LSUClient is doing, add the common `-Verbose` parameter to the cmdlets you run.
If you are intimately familiar with Lenovos package XML format, perhaps even the output of `Get-LSUpdate -Verbose -Debug` will be useful to you.

{{< details title="Example" open=false >}}

This is a minimal example implementation of this best practice for demonstration purposes:

```powershell {linenos=table}
Start-Transcript -LiteralPath "$env:TEMP\lsuclient_$(Get-Date -Format 'yyyyMMdd-HHmmss').log"

$updates = Get-LSUpdate -Verbose
Write-Host "$($updates.Count) updates found"

$i = 1
foreach ($update in $updates) {
    Write-Host "Installing update $i of $($updates.Count): $($update.Title)"
    Install-LSUpdate -Package $update -Verbose
    $i++
}

Stop-Transcript
```

{{< /details >}}

## 4. Keep $ErrorActionPreference set to Continue

Not every internal operation LSUClient carries out will and must neccesarily always succeed. Sometimes a file is
[in use by another process](https://github.com/jantari/LSUClient/issues/80) or [just missing from Lenovos servers](https://github.com/jantari/LSUClient/issues/37).

Errors like these often only affect a single package out of many, and if they are severe enough that a particular package
cannot be processed further then it will be skipped, but all other packages can still be processed normally. This is the
behavior of PowerShells' default `ErrorActionPreference` setting: `Continue`. LSUClient does not attempt to silence or hide
errors because they contain useful information about what went wrong and why. Particularly if you host your own package
repository, any repository-related failures for example are important to know about so you can address them.

So while it is a common best practice for PowerShell scripts, with `ErrorActionPreference` set to `Stop` one
of these "small" errors will always halt your entire script. Because of this I recommend against setting the
`ErrorActionPreference` to `Stop` in your LSUClient scripts.
