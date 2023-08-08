---
title: Uninstalling OneDrive from Windows 10
date: 2017-03-02 13:40:55 +0100
categories: [Software]
tags: [Windows]
image: onedrive-cover.jpg
img_path: /assets/img/posts/2017/
---

I don't need OneDrive, I don't want to use OneDrive, but I want Windows. For this reason I have written the following ~30 lines Batch script, to **wipe out OneDrive from any Windows 10 machine**.

Just save [RemoveOneDrive.bat](//static.fabioiotti.com/blog/windows/RemoveOneDrive.bat) script somewhere on your computer and run it with double click.

> WARNING: you might want to make a backup copy of any OneDrive not-yet-synchronized file before proceeding.

```batch
@echo off

echo Stopping OneDrive...
taskkill /f /im OneDrive.exe

echo Uninstalling OneDrive...
if %PROCESSOR_ARCHITECTURE%==x86 (
    %SystemRoot%\System32\OneDriveSetup.exe /uninstall
) else (
    %SystemRoot%\SysWOW64\OneDriveSetup.exe /uninstall
)

echo Removing folders...
rd "%UserProfile%\OneDrive" /Q /S
rd "%LocalAppData%\Microsoft\OneDrive" /Q /S
rd "%ProgramData%\Microsoft OneDrive" /Q /S
rd "C:\OneDriveTemp" /Q /S

echo Cleaning registry...
reg delete "HKEY_CLASSES_ROOT\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}" /f
reg delete "HKEY_CLASSES_ROOT\Wow6432Node\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}" /f

echo OK
pause
```

I have successfully run this script on every Windows 10 computer I own. Windows Update might reinstall OneDrive every few months, but it takes no time to run this script again.

> NOTE: if you change your mind you can reinstall OneDrive by running *OneDriveSetup.exe*. You can find it in system folder on your computer (no download required).

---

> References:

> https://techjourney.net/disable-or-uninstall-onedrive-completely-in-windows-10/
