---
layout: post
date: 2021-07-08
categories: debugging, windbg
title: List of WinDbg commands
---

## Help ##

`.hh {command}`
: show help for command

## Symbols and modules ##

`.sympath`
: show the symbol search path

`ld {module-name}`
: (re)load symbols for a module

`lm`
: list all modules

`lmvm {module-name}`
: dump information about a specific module

`x {module-name}!{function}`
: resolve a native function address, wildcards accepted

`ln {address}`
: find nearest symbol

## Processes and Threads ##

`|`
: processes list

`|{process-num}s`
: switch to another process

`.childdbg 1`
: enable child process debugging

`~`
: list all threads

`~{thread-num}s`
: switch to another thread

`~*{command}` or `~e!{ext-command}`
: execute command on all the threads [^6]

## Call stack ##

`k [{number of frames}]`
: show the native call stack

`~*k`
: show native call stacks for all threads

`!uniqstack`
: group threads call stacks and avoid duplicates

`.frame [/c] {frame-number}`
: switch to another call stack frame

## Variables ##

`dv [{pattern}]`
: display local variables, need to have private symbols

`? {expression}` and `?? {c++-expression}`
: evaluate an expression

`r [{registry-name}]`
: dump register value

## Memory ##

`d{format} {address}`
: dump memory in specific format

`db {address}`
: bytes and ascii

`dp {address}`
: pointer size

`dps {address}`
: display addresses and symbols referencing them [^1]

`dt {type} {address}`
: use specific type layout when dumping memory

`!address {address}`
: show details about a memory region to which the address belongs

`e{format} {address} {value(s)}`
: edit memory [^2]

## Program Execution ##

`u {address}`
: show disassembly

`uf {address}`
: disassemble a function

`g`
: continue

`gu`
: continue until function finishes

`t`
: step into code/assembly

`l+t`
: enable source stepping

`p`
: step over

`{g|t|p}a {address}`
: step until a specific address

`wt -l1 -oa -or`
: watch trace a function

## Debugging events ##

`sxe [-c {cmd1}] [-c2 {cmd2}] {exception|event}`
: enable a given exception or event [^3]

`sxd {exception|event}`
: enable only 2nd chance notification (unhandled exceptions)

`gn`
: mark exception as handled

## Breakpoints ##

`bl`
: list all breakpoints

`bp {address|function-name} [{command}]`
: create a breakpoint

`b{e|d} {breakpoint-number}`
: enable/disable breakpoint

`bc {breakpoint-number|*}`
: remove breakpoint

`[~{thread-num}]ba {access}{size} {address} [{command}]`
: create data breakpoint [^4]

## Visualisation ##

`dx {expression}`
: navigate through debugging objects [^5]

`.nvload {filename}`
: load type visualsation settings

## Managed Code ##

`.chain`
: show loaded extensions

`.loadby sos coreclr`
: load SOS for .Net core

`.loadby sos clr`
: load SOS for .Net Framework

`!sos.help {command}`
: SOS help

`!dumpdomain`
: list app domains

`!dumpmt {address}`
: dump a method table

`!dumpclass {address}`
: dump class detalis

`!dumpmd {address}`
: dump method descriptor

`!name2ee {module}!{type-or-method}`
: resolve a class name into method table, or method name into descriptor

`!bpmd {module} {method}`
: create a method breakpoint

`!bpmd -md {md}`
: create a method breakpoint

`!bpmd {source-file}:{line-number}`
: create a method breakpoint

`!bpmd -list`
: list breakpoints

`!bpmd -clear {breakpoint-number}`
: remove breakpoint

`!bpmd --clearall`
: remove all breakpoints

`!eeheap [-gc] [-loader]`
: show information of clr memory

`!dumpheap -stat`
: show managed heap stats

`!dumpheap -mt {mt}`
: dump objects of a method type

`!dumpheap -type {typename}`
: dump objects of a type

`!dumpobj {address}`
: dump a managed object

`!gcroot {address}`
: see references to an object

`finalizequeue`
: show objects registered for finalization

`!threads`
: list all managed threads

`!clrstack`
: show managed call stack of current thread

`!dumpstack`
: show complete call stack of current thread

`!dso`
: show managed objects in the call stack

`!pe`
: show exception details

`!dumpil {md}`
: dump IL code for a method

`!ip2md {address}`
: match assembly instruction with MD

`u {md|address}`
: disassemble a method or code address

## Examples ##

[^1] : `dps @rsp`

[^2] : `eb 0x123456 'n' 'e' 'i' 'l'`

[^3] : `sxe clr`

[^4] : `ba r4 0x12345`

[^5] : `dx -r1 Debugger.Sessions`

[^6] : `~*e!clrstack` - show all managed callstacks for all threads