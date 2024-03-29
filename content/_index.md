---
title: Introduction
type: docs
bookToc: false
asciinema: true
---

{{< center >}}
<h1 id="lsuclient" style="margin-top: 0; font-weight: bold;">LSUClient</h1>

{{< render-markdown >}}

![PowerShell Gallery](https://img.shields.io/powershellgallery/dt/LSUClient?label=PowerShell%20Gallery&logo=Powershell&logoColor=FFFFFF&style=flat)
![PowerShell Gallery Version](https://img.shields.io/powershellgallery/v/lsuclient?label=Latest&logo=powershell&logoColor=FFF)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/jantari/lsuclient?label=Latest&logo=github)

![logo](./logo_220px.png)

Orchestrate driver, BIOS/UEFI and firmware updates for Lenovo computers - with PowerShell!

{{< /render-markdown >}}
{{< /center >}}

{{< raw-html >}}
<div style="display:block;margin-left:auto;margin-right:auto" class="asciinema-sizer">
{{< /raw-html >}}

<br>&nbsp;
<br>

LSUClient is an all-PowerShell alternative to "Lenovo System Update" that gives you the control and flexibility to manage
driver and firmware updates like you've always wanted to.

## Installation

```powershell
Install-Module -Name 'LSUClient'
```

## Highlight Features

- Does driver, BIOS/UEFI, firmware and utility software updates
- Allows for fully silent and unattended update runs
- Work with updates and even their results as PowerShell objects to build any custom logic imaginable
- Fetch the latest updates directly from Lenovo or use an internal repository of your own for more control
- Can work alongside, but does not require Lenovo System Update or any other external program
- Run locally or manage/report on an entire fleet of computers remotely
- Full Web-Proxy support including authentication
- Supports not only business computers but consumer lines too (e.g. IdeaPad)
- Free, open-source and permissively licensed!

## Watch A Demo

{{< raw-html >}}
</div>
{{< /raw-html >}}

{{< asciinema-player key="lsuclient" >}}
