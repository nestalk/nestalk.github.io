---
layout: post
date: 2025-03-24
categories: tips, keyboard, vi
title: Remapping caps lock to escape
---

As someone that uses vi mode in editors access to the escape key is essential, but it is hard to reach. To make it easier to reach I remap a useless key to escape. And there is no more useless key than caps lock.

This powershell snippet will remap caps lock to escape. It needs to be run as administrator.

```powershell
$remap = New-Object -TypeName byte[] -ArgumentList 20
$remap[8] = 2
$remap[13] = 0x00
$remap[12] = 0x01
$remap[15] = 0x00
$remap[14] = 0x3a

$key = 'HKLM:\SYSTEM\CurrentControlSet\Control\Keyboard Layout'
New-ItemProperty -Path $key -Name 'Scancode Map' -Value $remap -Force
```