---
layout: post
date: 2021-07-09
categories: tips, debugging, c#
title: Setting the Symbol path
---

In order to set the symbol servers across different debuggers it is best to set the symbol path environment variable.

The variable name is `_NT_SYMBOL_PATH`, to set the local cache to a folder user the `cache*{folder}` option and for each symbol server you are adding use the `SRV*{server}` option. Each option is separated by a `;`.

Full example:

`cache*C:\Symbols;SRV*https://msdl.microsoft.com/download/symbols;SRV*https://symbols.nuget.org/download/symbols`
