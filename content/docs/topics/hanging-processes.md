---
title: Troubleshooting Hanging Processes
weight: 40
---

# Hanging processes

By nature, LSUClient has to run and wait on external processes such as hardware-detection routines or driver installers to complete.

The executables and installers Lenovo bundles in their packages are generally designed to run silently and without user interaction.
This means LSUClient can just execute these processes, wait for them to complete and then continue. However, it is unfortunately
sometimes the case that a process LSUClient is waiting on gets stuck indefinitely and never exits on its own.

Starting with Version 1.4.2, LSUClient will log warnings about any long-running processes at regular intervals so that
such problem cases can be easily identified.

If you run into this, you should proceed with these two options:

## Fixing hanging processes

It is always ideal to have a process finish successfully instead of hanging.

First, if you run LSUClient remotely or in an unattended scenario, make sure you are only installing
packages that can be installed silently and non-interactively in the first place as covered in the
[best practices article](/LSUClient-docs/docs/topics/examples/#install-only-packages-that-can-be-installed-silently-and-non-interactively).

Often when a process is stuck indefinitely, it is because it did not run entirely silently, opened up a graphical window and is waiting
for user input to continue. When this happens it can easily be seen by someone in front of the computer, but to also make identifying this
behavior remotely easier (without being in front of the computer), when a process is killed by LSUClient due to hitting a runtime limit,
LSUClient checks for any open windows and if there are any, captures and prints their text contents to the console. An example output of
this can be found [below](#example-of-a-package-installation-exceeding-the-maxinstallerruntime). The messages in the open windows often
explain or can be googled to understand why the problem occurred. Sometimes a process may also leave logfiles that can help troubleshoot
the cause of the hanging.

You can also search the issues on GitHub for the package name and keywords like "hang" or "stuck" to see if other people have had the same
problem. For example, [GitHub issue #49](https://github.com/jantari/LSUClient/issues/49) details a problem and solution with an Intel Graphics
Driver installer that, despite being run in silent mode, pops up an error message under certain conditions and then sits there indefinitely.

## Killing hanging processes

Starting with LSUClient 1.5.0, you can configure runtime limits for external processes. When a process exceeds the limit, it is forcefully terminated.
Killing a running process is not ideal, but as a last resort it can be a better option than having your entire LSUClient script halt indefinitely, especially in
unattended scenarios.

These time limits are configured with `Set-LSUClientConfiguration`, see the [`Set-LSUClientConfiguration` cmdlet help](/LSUClient-docs/docs/cmdlets/set-lsuclientconfiguration/)
for details and examples on how. You can get the current limit values with `Get-LSUClientConfiguration`. Some limits may be enabled by default.

{{< hint danger >}}
As a precaution LSUClient does not enforce the MaxInstallerRuntime setting for BIOS and firmware update installers.

However this automatic safeguard is based on package metadata such as Type, Category, Title etc. and therefore may not be perfect.
If you configure a MaxInstallerRuntime, always be aware of the risk of potentially forcefully stopping an in-progress firmware update!
{{< /hint >}}

When a package install process exceeds the `MaxInstallerRuntime` time limit and is killed, the `FailureReason` of the `PackageInstallResult` object returned
by `Install-LSUpdate` will be set to `PROCESS_KILLED_TIMELIMIT`. When this happens you should look into what caused the process to run for so long and whether
it can be fixed.

For example, reproducing a problem with an Intel Graphics Driver installer that was [reported on GitHub](https://github.com/jantari/LSUClient/issues/49)
now gets logged with the following output when it is killed:

### Example of a package installation exceeding the MaxInstallerRuntime

```plain
jantari@AMDESKTOP:~
└─ PS> Set-LSUClientConfiguration -MaxInstallerRuntime (New-TimeSpan -Minutes 6)
jantari@AMDESKTOP:~
└─ PS> Install-LSUpdate -Package $igfx -Path C:\Windows\Temp -Verbose
VERBOSE: Installing package n2hdr35w ...
WARNING: Process 'C:\Windows\Temp\n2hdr35w\installer.exe' has been running for 00:05:00.0588631
WARNING: Process has exceeded the configured runtime limit of 00:06:00
WARNING: Process has windows open, this can help troubleshoot why it timed out:
WARNING: - Title: -------------------------------------------
WARNING: HandledStartupErrorWindow
WARNING: - Content: -----------------------------------------
WARNING: Graphics Driver Installer
WARNING:
WARNING:
WARNING:
WARNING: The application cannot be launched.
WARNING: The application is in an unauthorized location. Please move to a non-system folder.
WARNING: OK
WARNING: ----------------------------------------------------
WARNING: Killing process 13104 'Installer' due to exceeding time limit ...


ID             : n2hdr35w
Title          : Intel Graphics Driver - 10 (1709 or Later)/11 (21H2 or Later)
Type           : Driver
Success        : False
FailureReason  : PROCESS_KILLED_TIMELIMIT
PendingAction  : NONE
ExitCode       : -1
StandardOutput :
StandardError  :
LogOutput      :
Runtime        : 00:05:59.3823498
```

