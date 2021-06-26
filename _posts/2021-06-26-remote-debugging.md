---
layout: post
date: 2021-06-26
categories: tips, debugging, c#
title: Remotely debugging IIS in Visual Studio
---

Sometimes you need to debug an IIS site on a remote server.

1. Install [Remote Tools for Visual Studio](https://visualstudio.microsoft.com/downloads/#remote-tools-for-visual-studio-2019) on the server
2. Open port 4024 so that you can access from your pc
3. Run Remote Debugger as an administrator
4. Open Visual Studio with a copy of the codebase
5. Click **Debug -> Attach to Process**
6. Enter the remote server name as the target
7. Click **Refresh**
8. Make sure **Show processes from all users** is selected
9. Select the process you want to debug and select Attach

See [this post]({% post_url 2018-06-11-getting-the-process-id-of-iis-app-pool %}) to get the id of the correct process.

The remote debugger will not pass the symbols from the remote server to the debugger which means your breakpoints will not be hit.

To load them

1. Copy the pdb files of the assemblies you want to debug to your local machine
2. In visual studio open the modules window while debugging, **Debug -> Windows -> Modules**
3. Right click on the module that isn't loaded and select **Load Symbols**
4. Select the pdb files you copied from the server

Once you select the pdb files to load your breakpoints will then be hit.
