---

title: " Windows FLS Gui tool "

categories:
  - Tool
tags:
  - Windows
  - Tool

---

# A little GUI FLS making Tool for Windows invests

The Sleuth Kit contain a tool name : FLS.

It's a command line tool and take args that are a pain in the ass to remember. Furthermore u need to make a Mactime afterward to get human readable timestamp.

So i decided to create my own and i wanted it to be simple and straightforward.

Here it is :[FLSGUITOOL](https://github.com/Xbloro/FLSGUI "here"). 

A compiled version is available one the release section.

You can still compile it yourself with MSbuild : 

```bash
MSBuild.exe /t:Publish /p:SelfContained=True /p:Configuration=Release /p:Plateform=x86 /p:PublishDir=C:\Users\WHEREVERUWANTTOPUBLISHIT
```



## How to 

Since it skips files that it isn't allowed to read, i suggest to run it as an admin.

So basically,  u just have to click : 

Open the binary : 

![welcome](/assets/images/GuiFLS/flsgui_welcoming.png?raw=true "welcome")

click on the button and select the path to save the csv result file and the path where to begin the FLS.

![welcome](/assets/images/GuiFLS/select_path.png?raw=true "selectpath")

One it's launch, the waiting screen is showing

![welcome](/assets/images/GuiFLS/wait_screen.png?raw=true "wait")

Once it's done, the result file look like that : 

![welcome](/assets/images/GuiFLS/result.png?raw=true "results")

Tadaaa.

i'll maybe do the same without gui but FLS already exist so ...

Hope it helps.
