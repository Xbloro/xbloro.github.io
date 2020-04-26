---

title: " Tool you Must Have on a DFIR "

categories:
  - Tool
tags:
  - Windows
  - Tool

---

# curently editing 

# Tool you must have in a DFIR on Windows

When investigating on a Windows system, their is a lot of things to check. Furthermore, in real condition, their are lot of machine to check and this can take you a lot of time.

Time is precious during an attack !

Some tools are very helpful and can save you a lot of time. 

You 'll find a non exhaustive list of must have tools below. In every DFIR, i have them ready on an USB key.

I suggest that you rename them to be able to recognize them directly when investigating

## SysInternal

The SysInternal suite are some tools developed by Microsoft.

[Find them here](https://docs.microsoft.com/en-us/sysinternals/)

You can find the most interesting one below.

Note that their is a lot of them not mentioned that worth a look.

### Autoruns

This one is pretty obvious and let you see what process are launching at startup.

Be sure to look at Service, Drivers(for rootkit), Logon,

Their is a console mode name autorunsc that let you check process on virusTotal.

### TCPVIEW

Get all TCP connection, pretty useful when looking for a connection to a CNC.

### Process Explorer

*Process Explorer* shows you information about which handles and DLLs processes have opened or loaded.

 It can create memory dump of a process that can be debug or RE.

### Process Monitor

*Process Monitor* is an advanced monitoring tool for Windows that shows real-time file system, Registry and process/thread activity

### Last Activity View

Let you see the last activity made on the machine.

## The SleuthKit

### Mactime

Fls create a timeline of the file of the machine. This allows you to see when a file was written or last accessed.

This is very helpful to look for compromised file.

### FLS

A Mactime parser that create more friendly view in CSV.

### AMCACHE Parser

Let you get the content of the Amcache witch list program's installation details.

### RegRipper

A registry Key parser.