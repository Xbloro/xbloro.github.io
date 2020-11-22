---

title: " Tools that can be realy helpful on a DFIR "

categories:
  - Tool
tags:
  - Windows
  - Tool

---

# Nice tools to use in a live DFIR on Windows

When investigating on a Windows system, their is a lot of things to check. Furthermore, in real condition, a big infrastructure to invest is realy time consuming.

Time is precious during an attack !

Some tools are very helpful and can save you a lot of time. 

You 'll find a non exhaustive list of must have tools below. In every DFIR, i have them ready on an USB key.

I suggest that you rename them to be able to recognize them directly when investigating. 

---



## SysInternal

The SysInternal suite are some tools developed by Mark Russinovich's.

[Find them here](https://docs.microsoft.com/en-us/sysinternals/)

You can find the most interesting one below.

Note that their is a lot of them not mentioned that worth a look.

### Autoruns

This one is pretty obvious and let you see what process are launching at startup.

Be sure to look at Service, Drivers(for rootkit), Logon,

Their is a console mode name autorunsc that let you check process on virusTotal.

![alt text](/assets/images/niceTools/autoruns.png?raw=true "Autoruns")  

### TCPView

Get all TCP connection, pretty useful when looking for a connection to a CNC.

![alt text](/assets/images/niceTools/tcpview.png?raw=true "TCPView")  

### Process Explorer

*Process Explorer* shows you information about which handles and DLLs processes have opened or loaded.

It can create memory dump of a process that can be debug or RE.

![alt text](/assets/images/niceTools/psexplo.png?raw=true "ProcessExplorer")  

### Process Monitor

![alt text](/assets/images/niceTools/procmon.png?raw=true "ProcessMonitor")  

*Process Monitor* is an advanced monitoring tool for Windows that shows real-time file system, Registry and process/thread activity



---

## Nirsoft

Nirsoft produce nice dfir tool, you can [find theme here](https://www.nirsoft.net/utils/index.html)

### Last Activity View

Let you see the last activity made on the machine.

---



## The SleuthKit 

[Get it there](https://www.sleuthkit.org/)

### Mactime

Fls create a timeline of the file of the machine. This allows you to see when a file was written or last accessed.

This is very helpful to look for compromised file.

### FLS

A Mactime parser that create more friendly view in CSV.

![alt text](/assets/images/niceTools/tsk.png?raw=true "tsk")  

---

## REDLINE

[Dowload it](https://www.fireeye.com/services/freeware/redline.html) (you have to take a survey first)

Very powerfull dead forensic tool. It allows you to create a payload to gather useful data (memory dump, ioc search, system info and more). 

Be sure to get a payload ready on your USB KEY.

![alt text](/assets/images/niceTools/payload1.png?raw=true "Payload")  

---

## Live OS

I don't like to use liveOperating system in DFIR because you have to shutdown the guest and restart it etc etc. I'd better do a copy for DEAD forensic.

 However it can be useful sometimes.

Tsurugi is a debian based system made for forensic [get it there ](https://tsurugi-linux.org/)

Kali that i don't need to present [here it is](https://www.kali.org/)

![alt text](/assets/images/niceTools/tsu.png?raw=true "Tsurugi")  

## Other 

### Loki

A tool to look for IOC >> [here](https://github.com/Neo23x0/Loki)

Be careful with signatures, some AV can flag them.

![alt text](/assets/images/niceTools/loki.png?raw=true "loki")  

### Eric Zimmerman tools

[Made by Eric Zimmerman](https://ericzimmerman.github.io/#!index.md)

Amcache Parser let you get the content of the Amcache witch list program's installation details.

I suggest to check ntuser.dat and the shimcash (app compat cache)

You can find [here](https://www.ssi.gouv.fr/uploads/2019/01/anssi-coriin_2019-analysis_amcache.pdf) a very nice article about it and how to get useful informations from it.

### RegRipper

find it [there](https://github.com/keydet89/RegRipper2.8)

A registry Key parser. Some advices about it >> [here](https://tools.kali.org/forensics/regripper)

---

