---
title: Introduction
type: docs
bookToc: false
asciinema: true
---

{{< center >}}

# LSUClient

![PowerShell Gallery](https://img.shields.io/powershellgallery/dt/LSUClient?label=PowerShell%20Gallery&logo=Powershell&logoColor=FFFFFF&style=flat)
![PowerShell Gallery Version](https://img.shields.io/powershellgallery/v/lsuclient?label=Latest&logo=powershell&logoColor=FFF)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/jantari/lsuclient?label=Latest&logo=github)

![logo](./logo_220px.png)

A PowerShell module that partially reimplements the "Lenovo System Update" program for convenient,
automatable and worry-free driver and system updates for Lenovo computers.

{{< /center >}}

```powershell
Install-Module -Name 'LSUClient'
```

{{< asciinema-player key="lsuclient" >}}

## Highlight features

- Does driver, BIOS/UEFI, firmware and utility software updates
- Allows for fully silent and unattended update runs
- Fetch the latest updates from Lenovo directly or use an internal repository of your own for more control
- Can work alongside, but does not depend on or use Lenovo System Update or any other external programs
- Work with updates and even their results as PowerShell objects to build any custom logic imaginable
- Run locally or manage/report on an entire fleet of computers remotely
- Full Web-Proxy support including authentication
- Supports not only business computers but consumer lines too (e.g. IdeaPad)
- Free, open-source and permissively licensed!
