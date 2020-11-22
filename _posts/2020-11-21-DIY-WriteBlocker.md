---

title: " Raspi DIY Imager "

categories:
  - Tool
tags:
  - DFIR
  - Tool

---

# Creating a  WriteBlocker with a  RaspBerrry

I  was bored in the lockdown and i found an old PC of mine. I thought  that i could make a forensic scenario with it.

So First off all, i needed to copy the disk. But, i'm not at work and i don't have any forensic duplicator / Write-Blocker tableau. And trust me, this shit cost so much money, even the basics one. So i decided to make my own using my Raspberry pi 3.

I used Raspian but don't, tools like FTK are not build for ARM. Instead u may use [Kali ARM](https://www.kali.org/docs/arm/kali-linux-raspberry-pi/. "kali for raspberry"), FTK is built in.

I tried Tsurugi Linux but it was lagging to much.

---

## Choosing the method

I don't have any hardware things related to USB etc. so i can't create an analogical one. It will be software based.

It's really not recommended to use  a software blocker because they are still writing things, for example ext 4 will replay the journal (and thus write to the partition) even when mounted "read only". See >>  [ext4 documentation](https://www.kernel.org/doc/Documentation/filesystems/ext4.txt "ext4 documentation")

But hey, it's not like i have a choice and it will be used to my own project, not at work, so it will do.

First of all we need to make sure that our evidence support will be mounted as ReadOnly.

Their are  many ways to achieve this : 

- option 1 : Mount as RW only white-listed USB and mountig the rest as RO;
- option 2 : Not mounting anythings automatically;
- option 3 : Block only one USB port as RO so everything that's plug in it is mounted as RO.

My first choice was the option 3 but i could never figure out how to do that  ¯\_(ツ)_/¯ so if you have a way, please tell me !

I went with option 2.

## Canceling auto USB mountig 

we need  to edit this : 

```bash
sudo nano ~/.config/pcmanfm/LXDE-pi/pcmanfm.conf
```

and change : 

```bash
[volume]
mount_on_startup=1
mount_removable=1
```

to

```bash
[volume]
mount_on_startup=0
mount_removable=0
```

And Tadaa, pretty ez right ? 

## Choosing a forensic software duplicator

I'm used to [FTK imager](https://accessdata.com/product-download/debian-and-ubuntu-x64-3-1-1 "ftk") , it create E0 files which are legally admissible unlike iso format.

I also found [Johntcw imager](https://github.com/johntcw/Forensic-Imager "Johntcw imager"), a python  based imager with a nice GUI.

If you realy like EWF files and u should, [EWFacquire](https://linux.die.net/man/1/ewfacquire "ewfacquire") is made for you ! 

Let's try both ! 

## Testing

So let's test this badboy out.

As you can see when i plug an usb key in, it doesn't mount

![usbNotmounted](/assets/images/raspImager/ntmounted.png?raw=true "not mounted")

![usb](/assets/images/raspImager/usbPlug.png?raw=true "usbPluged")

we need to mount it as readOnly. Be sure to activate the flag "noload" to suppresses the loading of the journal and prevent writing on the disk by the system.

> "`noload` is mostly useful to mount a disk as read-only without changing it in the slightest way, not even replaying the journal. "

First we create a folder to mount the USBkey : 

```bash
sudo mkdir /media/usbFolder
```

Then we mount our key in it : 

```bash
sudo mount -ro,noload /dev/sda1 /media/usbFolder
```

As you can see, i can't write on the disk : 

![errWrite](/assets/images/raspImager/trycpy.png?raw=true "errWrite")

No it's time to image : 

I first used the GUI described before but it was crashing a soon as i plugged an USB key. I tried then FTK imager. But it wasn't build for ARM so it didn't work either...

We have 2 solution here :

- 1st one is to copy with DD.

- Second one is to Install Kali for Raspberry pi and use FTK imager or DD.

Since i plane to use my raspberry pi for other means i stayed with Raspian and used DD to copy my disk.

```bash
dd if=/dev/sdx of=whatever.iso bs=<block size> count=<volume size> status=progress
```

With EWF acquire : 

```bash
ewfacquire /dev/sdx
```

It will ask you lot of question like the investigator name, case name, etc., etc., because it's a forensic purpose tool.

 If u planed to use it full time i recommend to use Kali.

Tadaa, hope it helps.



