---

title: "criminal forensic invest "

categories:
  - DFIR
tags:
  - DFIR
  - Windows
---

I found my old laptop  in my parent's home while i was back there for Chrismas. I tough that it could be fun to make a little investigation on it. Using VM is pretty handy but they are empty of data, so things are to easy to find. Moreover, playing with screwdrivers and HDD add a nice fealing to the... experience(? lol).

# Scenario

U 're a Police / GOV / what ever arrest bad guys  Certified Forensic Examiner and the DEA  juste gave you the laptop used by a dealer.

The rapport say that the criminal was using it when they breached out. As soon as he saws them, he smashed the computer into the ground.

 The Detective in charge of the case want to take down the whole criminal network. He expects from you to recovers usefull informations like a list of contact or some rendez-vous with his provider.

# Acquisition 

Ok time to play. 

First of all, let's strike a pose. Like a prisoner we take picture of the evidence : 

Front : 

![Front](/assets/images/CrimForensic/laptopFront.png?raw=true "laptop front")

Back : 

![Back](/assets/images/CrimForensic/laptopBack.png?raw=true "laptop back")

Then we disembowel it to get the Drive. Screwdrivers on !

![HDDinside](/assets/images/CrimForensic/hddIn.png?raw=true "drive Insinde")

Because i'm not at work, i dont have acces to my forensics gear so i will make the disk copy using my [Raspi imager](https://xbloro.github.io/tool/DIY-WriteBlocker/), but w'ill just pretend it's OK ;)

 ![aquisition](/assets/images/CrimForensic/Aquisition.png?raw=true "aquiring")

After a long time, it's done. Time to investigate !

# Preliminary Investigation

When investigating on a machine, the first step is to recovers machine and users informations.

## Machine infos 

Using powershell would take 1 command but since it's dead forensic we need to search on different places to gather what we need

Since we need to look in registry keys, we have 2 options :  mount them in the windows reg key eddit, or parse them. 

The WinApi is pretty good if u want to create ur own parsing  tool. I have mine written in C#.

 [regRipper](https://github.com/keydet89/RegRipper3.0) work's well but it does'nt provide everythings.

Autospy parse them in it's laste version : 

 ![Parsed](/assets/images/CrimForensic/regParsedAutopsy?raw=true "Parsed")

Let's go to "C:\windows\system32\config" to dump our registry keys. We w'll juste take Software and System for now.



### OS information

Autopsy got a tab for OS name : 

 ![OsName](/assets/images/CrimForensic/osName.png?raw=true "osName")

We could also look for it in the "SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion" registry key.

We got way more info as u can see: 

 ![osInfoReg](/assets/images/CrimForensic/osInfoReg.png?raw=true "osInfoReg")

We need, the OS Name, the build, the version and the install date.



### TimeZone 

It's important to get the timeZone info in order to correlate corectly informations.

U can find it here :  "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation"

 ![timeZone](/assets/images/CrimForensic/timezone.png?raw=true "timezone")

This is **Romance Standard** Time so : 

This time zone supports daylight savings time.
The daylight name is: Romance Daylight Time.
This time zone covers the following region and/or area: (UTC+01:00) Brussels, Copenhagen, Madrid, Paris.



## Users 

 Autospy got a tab for it but it does not provide anything this time, let check out this key :

"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList"

U could alsow check the "C:\users" directory, but it might not be reliable.

We have 3 profile here, 2 not counting the guest one.

![profiles](/assets/images/CrimForensic/profileListe.png?raw=true "profiles")

# Investigation 

Ok we got every info we needed so let's check out if we can find interesting things for our comrad Detective.

As u can see, we got a lot of data here :) unlike a VM haha 

![datas](/assets/images/CrimForensic/datas.png?raw=true "datas")

According to the infos, our guy was on the computeur when they breached in. He might have not got enought time to hide/destroy correctly the evidences.

Lets check the NTUSER.dat file  

> Every time you make a change to the look and behavior of Windows and installed programs, whether that’s your desktop background, monitor resolution, or even which printer is the default, Windows needs to remember your preferences the next time it loads.

We are Looking for ShellBags :

Nice SANS [article](https://www.sans.org/reading-room/whitepapers/forensics/windows-shellbag-forensics-in-depth-34545) about shellbags.

> Microsoft Windows records the view preferences of folders and Desktop. Therefore, when the folder/Desktop is visited again, Windows can remember the location of the folder, view and positions of items. Microsoft Windows store the view preferences in the registry keys and values known as “ShellBags”.
>
> ShellBag information is crucial when forensicators need to know when and which folder a user accessed. For instance, when a company suspects an employee leaked a confidential document stored on the network, that employee’s computer may have the ShellBag information that demonstrates the folder containing that document was accessed shortly before the document was leaked. Furthermore, ShellBags may also show the folders or servers that employee should not access. 

Autospsy as handy as always, has got a tab for it :)

Lets check out what files were last accessed :

![founded](/assets/images/CrimForensic/founded.png?raw=true "founded")

We can see that the file "client.zip.aes" and the binary "GuiEncryption.exe" were last accessed 2 minutes before the breach of specOps.

That what we are looking for ! Lets extracts them. (Look for their path in the NTUSER.DAT entry and go with the file explorer and then right click -> extracts files).

Since we are playing with a binary, i suggest that u drop it in a excluded zone from ur AV and zip+pass it immediatly. We will execute it in a VM.

Let's go with Flare VM :)

The binary looks like, as it name sugest, an aes encryption tool.

Using peBear we can see that he have a dependency called "mscorlib", one of .net base class libraries that every program in C# depends on it. That mean our PE is a .net. That will be ez to RE.

We can use the tool [Detect it eazy](http://ntinfo.biz/index.html) as well : 

![die](/assets/images/CrimForensic/die.png?raw=true "die")

Let fire up dot peak : 

![petree](/assets/images/CrimForensic/petree.png?raw=true "petree")

We can see that it actually crypt file using AES and RSA.

But we do need the AES key.

This is not a RE tuto and i suck at RE but after searching the strings files we find a built in aes key !

![searchStr](/assets/images/CrimForensic/stringkeysearch.png?raw=true "searchstr") aeskeyBin

![keyIn](/assets/images/CrimForensic/aeskeyBin.png?raw=true "keyIn")

Since the attacker used this tool, let's try do decrypt the datas we found earlier with it.

The soft ask for a txt file so w'ill copy the key found in one :

![trykey](/assets/images/CrimForensic/trykey.png?raw=true "trykey")

Annnd, it failed that's not the good key :'(

Back to work.

The attacker was in a rush while encrypting so he might have not got enough time to destroy the key, lets search for it :

Autopsy got a delet file tab but their is a lot of elements in there, it will take time to proceed. Instead lets try to think like the guy.

The key is a file, since datas and software were found on his desktop, they is a great chance that the key is in the same place.

Sadly we didn't found the key here. Maybe she was there but deleted. Autopsy can carve deleted file and no data were written on the disk so it should be able to recovers it, if it exist tho. But before that lets think. What do we do when we delet file in windows ? 

 The first reflex is to add it to the trashBin right ?

Trash file is located in C:\ $RecycleBin, it is organise by users.

![recycle](/assets/images/CrimForensic/recycle.png?raw=true "recycle")

Our user have the uid 1000, lets search for txt files  with creation date near the one of the encrypted file : 

![keyfound](/assets/images/CrimForensic/keyfound.png?raw=true "keyfound")

![keyinfo](/assets/images/CrimForensic/keyInfo.png?raw=true "keyinfo")

Looks like we have our key 

Time to decrypt ! 

![works](/assets/images/CrimForensic/finito.png?raw=true "works")



![done](/assets/images/CrimForensic/datasuncrypted.png?raw=true "done")

Yeah we have got what our boss asked for.

Time to get a well deserved Donut.



## End

I hope you enjoyed this little scenario even if it's a fake one. It was fun to make for me.

Hope you learned some things too.

bisous





 

 









