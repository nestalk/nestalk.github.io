---
layout: post
date: 2018-06-11
categories: tips, iis
title: Getting the process id of an IIS application pool
---

I work on a system where there are multiple sites running on IIS, but although they run in seperate app pools, they use the same user account.

This causes a problem when debugging in Visual Studio as when choosing which process to debug, you only see the process id and username of the processes.

To find the process id's of each app pool, run the following command in an administrator command prompt:

```
C:\windows\System32\inetsrv\appcmd list wp
```

This will show you the process id and app pool name of all running IIS processes. You cant then select the correct process in VS.