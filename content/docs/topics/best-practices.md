---
weight: 20
---

# Best Practices

## 1. Search for and install updates a few times in a loop

Some packages and updates depend on others already being installed, so they might not show up during the first search for updates.
If you want to make sure "everything" has been installed and/or updated it is best to re-run 
`Get-LSUpdate` and `Install-LSUpdate` a few times or until no more updates are found.

## 2. Download all packages before starting to install any

Installing a network (NIC, WiFi adapter) driver can cause a short loss of network connectivity. When you feed packages found by `Get-LSUpdate` directly to `Install-LSUpdate` they will each be individually downloaded and then installed, therefore a network driver installation can cause the following package downloads to fail. Downloading all packages you intend to install beforehand with `Save-LSUpdate` will prevent this potential problem.

{{< hint info >}}
If you download packages to a custom location with `Save-LSUpdate -Path` you have to pass the same path to `Install-LSUpdate -Path` for it to find the previously downloaded files. If `Install-LSUpdate` cannot find the expected files locally it will still attempt to fetch them from the repository.
{{< /hint >}}

## 3. Write your own logfile

LSUClient is a PowerShell module, so you will utilize it by scripting around the functionality it exposes, adding your own logic, filtering,
workarounds or special cases according to your requirements. Unexpected behavior and errors can occur both within your scripts and within the
functions from LSUClient, so a good logfile that details what is happening and allows you to find the source of a problem in retrospect is a must.

If you don't know where to start, start with [`Start-Transcript`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript?view=powershell-5.1) to log whatever is being output to the console.
If you would like more insight into what LSUClient is doing, add the common `-Verbose` parameter to the cmdlets you run.
If you are intimately familiar with Lenovos package XML format, perhaps even the output of `Get-LSUpdate -Verbose -Debug` will be useful to you.
