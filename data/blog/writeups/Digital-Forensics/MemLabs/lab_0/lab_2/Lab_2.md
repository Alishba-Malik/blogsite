---

title: MemLab Series-Lab 2
date: '2024-07-27'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Given memory dump, Recover data from activist's memory dump..

---
# **MemLabs Lab 2 - A New World**

**Challenge description**

One of the clients of our company, lost the access to his system due to an unknown error. He is supposedly a very popular "environmental" activist. As a part of the investigation, he told us that his go to applications are browsers, his password managers etc. We hope that you can dig into this memory dump and find his important stuff and give it back to us.

**Note**: This challenge is composed of 3 flags.

**Challenge file**: [MemLabs_Lab2](https://mega.nz/#!ChoDHaja!1XvuQd49c7-7kgJvPXIEAst-NXi8L3ggwienE1uoZTk)

Before stepping on the challenge directly, let’s have a look on the challenge description. The emphasize on “environmental” is clearly giving the hint of environmental variables. Moreover, browsers and password manager are also mentioned in the description.Now, let’s give it a kick.

I’ll start by un-zipping the zipped file.

```bash
 7z x MemLabs-Lab2.7z 
```

Then like a old custom tradition, i’ll run the `imageinfo` plugin on the memory dump to know about the profiles.  

```bash
volatility -f MemoryDump_Lab2.raw imageinfo 
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img1.png)

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 kdbgscan
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img2.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img3.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img4.png)

Now let’s get into the real stuff, i mean hacking. Let’s list all the active processes using the command:

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 pslist
```

Let’s see what i got from all the processes

```bash
└─$ volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 pslist
Volatility Foundation Volatility Framework 2.6
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa8000ca0040 System                    4      0     80      541 ------      0 2019-12-14 10:35:21 UTC+0000                                 
0xfffffa80014976c0 smss.exe                248      4      3       37 ------      0 2019-12-14 10:35:21 UTC+0000                                 
0xfffffa80014fdb30 csrss.exe               320    312     10      446      0      0 2019-12-14 10:35:27 UTC+0000                                 
0xfffffa8001c40060 csrss.exe               368    360      8      237      1      0 2019-12-14 10:35:28 UTC+0000                                 
0xfffffa8000ca8840 psxss.exe               376    248     18      786      0      0 2019-12-14 10:35:28 UTC+0000                                 
0xfffffa8001c5a700 winlogon.exe            416    360      6      112      1      0 2019-12-14 10:35:30 UTC+0000                                 
0xfffffa8001c5b2b0 wininit.exe             424    312      3       75      0      0 2019-12-14 10:35:30 UTC+0000                                 
0xfffffa8001c95320 services.exe            484    424      8      206      0      0 2019-12-14 10:35:31 UTC+0000                                 
0xfffffa8001c9d910 lsass.exe               492    424      8      546      0      0 2019-12-14 10:35:31 UTC+0000                                 
0xfffffa8001c9e2d0 lsm.exe                 500    424     10      181      0      0 2019-12-14 10:35:31 UTC+0000                                 
0xfffffa8001cec790 svchost.exe             588    484     12      354      0      0 2019-12-14 10:35:35 UTC+0000                                 
0xfffffa8001d13060 VBoxService.ex          652    484     14      135      0      0 2019-12-14 10:35:36 UTC+0000                                 
0xfffffa8001d4ab30 svchost.exe             720    484      7      275      0      0 2019-12-14 10:35:37 UTC+0000                                 
0xfffffa8001d76320 svchost.exe             812    484     21      474      0      0 2019-12-14 10:35:38 UTC+0000                                 
0xfffffa8001da6930 svchost.exe             852    484     20      417      0      0 2019-12-14 10:35:38 UTC+0000                                 
0xfffffa8001dacb30 svchost.exe             876    484     39      962      0      0 2019-12-14 10:35:38 UTC+0000                                 
0xfffffa8001df65f0 audiodg.exe             268    812      7      131      0      0 2019-12-14 10:35:41 UTC+0000                                 
0xfffffa8001e1eb30 svchost.exe             472    484     12      301      0      0 2019-12-14 10:35:42 UTC+0000                                 
0xfffffa8001e47740 svchost.exe            1044    484     16      361      0      0 2019-12-14 10:35:43 UTC+0000                                 
0xfffffa8000cf9220 spoolsv.exe            1208    484     13      279      0      0 2019-12-14 10:35:47 UTC+0000                                 
0xfffffa8001ed8b30 svchost.exe            1248    484     18      303      0      0 2019-12-14 10:35:49 UTC+0000                                 
0xfffffa8001f4eb30 svchost.exe            1368    484     23      314      0      0 2019-12-14 10:35:51 UTC+0000                                 
0xfffffa8001f7c060 TCPSVCS.EXE            1412    484      4       97      0      0 2019-12-14 10:35:53 UTC+0000                                 
0xfffffa80020f2b30 taskhost.exe           1928    484      9      154      1      0 2019-12-14 10:36:04 UTC+0000                                 
0xfffffa8002105b30 taskeng.exe            1996    876      5       79      0      0 2019-12-14 10:36:04 UTC+0000                                 
0xfffffa800211e7c0 dwm.exe                2012    852      4       72      1      0 2019-12-14 10:36:04 UTC+0000                                 
0xfffffa8002131340 explorer.exe           1064   2004     37      989      1      0 2019-12-14 10:36:05 UTC+0000                                 
0xfffffa80021bb240 SearchIndexer.         1836    484     12      628      0      0 2019-12-14 10:36:11 UTC+0000                                 
0xfffffa80021c0b30 VBoxTray.exe           1896   1064     13      138      1      0 2019-12-14 10:36:13 UTC+0000                                 
0xfffffa80022a3b30 csrss.exe              2308   2300      9      246      2      0 2019-12-14 10:36:24 UTC+0000                                 
0xfffffa80022a73e0 winlogon.exe           2336   2300      4      109      2      0 2019-12-14 10:36:24 UTC+0000                                 
0xfffffa8000e76060 taskhost.exe           2604    484      9      154      2      0 2019-12-14 10:36:29 UTC+0000                                 
0xfffffa8000e95060 dwm.exe                2652    852      3       71      2      0 2019-12-14 10:36:29 UTC+0000                                 
0xfffffa8000e9a110 explorer.exe           2664   2632     19      632      2      0 2019-12-14 10:36:29 UTC+0000                                 
0xfffffa8000edcb30 VBoxTray.exe           2792   2664     12      139      2      0 2019-12-14 10:36:30 UTC+0000                                 
0xfffffa80022e5950 cmd.exe                2096   2664      1       19      2      0 2019-12-14 10:36:35 UTC+0000                                 
0xfffffa8000e63060 conhost.exe            2068   2308      2       50      2      0 2019-12-14 10:36:35 UTC+0000                                 
0xfffffa8002109b30 chrome.exe             2296   2664     27      658      2      0 2019-12-14 10:36:45 UTC+0000                                 
0xfffffa8001cc7a90 chrome.exe             2304   2296      8       71      2      0 2019-12-14 10:36:45 UTC+0000                                 
0xfffffa8000eea7a0 chrome.exe             2476   2296      2       55      2      0 2019-12-14 10:36:46 UTC+0000                                 
0xfffffa8000ea2b30 chrome.exe             2964   2296     13      295      2      0 2019-12-14 10:36:47 UTC+0000                                 
0xfffffa8000fae6a0 chrome.exe             2572   2296      8      177      2      0 2019-12-14 10:36:56 UTC+0000                                 
0xfffffa800105c060 WmiPrvSE.exe           2636    588     12      293      0      0 2019-12-14 10:37:02 UTC+0000                                 
0xfffffa800100c060 WmiApSrv.exe           2004    484      6      115      0      0 2019-12-14 10:37:05 UTC+0000                                 
0xfffffa800230eb30 chrome.exe             1632   2296     14      219      2      0 2019-12-14 10:37:12 UTC+0000                                 
0xfffffa800101e640 dllhost.exe            2376    588      9      250      1      0 2019-12-14 10:37:40 UTC+0000                                 
0xfffffa800224a8c0 KeePass.exe            3008   1064     12      316      1      0 2019-12-14 10:37:56 UTC+0000                                 
0xfffffa8002230b30 sppsvc.exe             2764    484      5      151      0      0 2019-12-14 10:38:00 UTC+0000                                 
0xfffffa80010e5b30 svchost.exe            1076    484     17      337      0      0 2019-12-14 10:38:02 UTC+0000                                 
0xfffffa80010f44a0 wmpnetwk.exe            928    484     18      523      0      0 2019-12-14 10:38:03 UTC+0000                                 
0xfffffa80011956a0 notepad.exe            3260   3180      1       61      1      0 2019-12-14 10:38:20 UTC+0000                                 
0xfffffa80011aa060 DumpIt.exe             3844   1064      2       45      1      1 2019-12-14 10:38:43 UTC+0000                                 
0xfffffa8001194570 conhost.exe            3852    368      2       52      1      0 2019-12-14 10:38:43 UTC+0000                                 
0xfffffa8001189b30 WmiPrvSE.exe           4004    588      9  1572864 ------      0 2019-12-14 10:39:00 UTC+0000                                 

```

I find a couple of red flags here, like:

- KeePass.exe
- chrome.exe
- notepad.ex
- cmd.exe
- explorer.exe

and many more.

But as i have mentioned the hints (browsers and password manager) already, so without getting distracted, let’s follow the track, until we find a dead end.

Let’s start from the first hint `environmental variables` and find out where it gets me.

Let’s explore environmental variables

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 envars
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img10.png)

What I found sus is this string `ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9` , but when i decoded it i got my first flag.

```bash
echo -n "ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9" | base64 -d
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img11.png)

### Flag 1:

```bash
flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}
```

Now let’s dig into `KeePass.exe` (i-e passwords, the next hint). Let’s check what arguments are passed with this process and i’ll use the `cmdline` plugin for this one.It’ll list all the arguments passed for all the processes.

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 cmdline
```

Particularly focusing on `KeePass.exe` , this is what i found:

```bash
$ volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 cmdline
Volatility Foundation Volatility Framework 2.6
************************************************************************
System pid:      4
************************************************************************
smss.exe pid:    248
Command line : \SystemRoot\System32\smss.exe
************************************************************************
csrss.exe pid:    320
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
************************************************************************
csrss.exe pid:    368
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
************************************************************************
psxss.exe pid:    376
Command line : %SystemRoot%\system32\psxss.exe
************************************************************************
winlogon.exe pid:    416
Command line : winlogon.exe
************************************************************************
wininit.exe pid:    424
Command line : wininit.exe
************************************************************************
services.exe pid:    484
Command line : C:\Windows\system32\services.exe
************************************************************************
lsass.exe pid:    492
Command line : C:\Windows\system32\lsass.exe
************************************************************************
lsm.exe pid:    500
Command line : C:\Windows\system32\lsm.exe
************************************************************************
svchost.exe pid:    588
Command line : C:\Windows\system32\svchost.exe -k DcomLaunch
************************************************************************
VBoxService.ex pid:    652
Command line : C:\Windows\System32\VBoxService.exe
************************************************************************
svchost.exe pid:    720
Command line : C:\Windows\system32\svchost.exe -k RPCSS
************************************************************************
svchost.exe pid:    812
Command line : C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
************************************************************************
svchost.exe pid:    852
Command line : C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted
************************************************************************
svchost.exe pid:    876
Command line : C:\Windows\system32\svchost.exe -k netsvcs
************************************************************************
audiodg.exe pid:    268
Command line : C:\Windows\system32\AUDIODG.EXE 0x2a4
************************************************************************
svchost.exe pid:    472
Command line : C:\Windows\system32\svchost.exe -k LocalService
************************************************************************
svchost.exe pid:   1044
Command line : C:\Windows\system32\svchost.exe -k NetworkService
************************************************************************
spoolsv.exe pid:   1208
Command line : C:\Windows\System32\spoolsv.exe
************************************************************************
svchost.exe pid:   1248
Command line : C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork
************************************************************************
svchost.exe pid:   1368
Command line : C:\Windows\system32\svchost.exe -k LocalServiceAndNoImpersonation
************************************************************************
TCPSVCS.EXE pid:   1412
Command line : C:\Windows\System32\tcpsvcs.exe
************************************************************************
taskhost.exe pid:   1928
Command line : "taskhost.exe"
************************************************************************
taskeng.exe pid:   1996
Command line : taskeng.exe {5C750F66-3822-47A6-A873-1EB5DDC99795}
************************************************************************
dwm.exe pid:   2012
Command line : "C:\Windows\system32\Dwm.exe"
************************************************************************
explorer.exe pid:   1064
Command line : C:\Windows\Explorer.EXE
************************************************************************
SearchIndexer. pid:   1836
Command line : C:\Windows\system32\SearchIndexer.exe /Embedding
************************************************************************
VBoxTray.exe pid:   1896
Command line : "C:\Windows\System32\VBoxTray.exe" 
************************************************************************
csrss.exe pid:   2308
Command line : %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16
************************************************************************
winlogon.exe pid:   2336
Command line : winlogon.exe
************************************************************************
taskhost.exe pid:   2604
Command line : "taskhost.exe"
************************************************************************
dwm.exe pid:   2652
Command line : "C:\Windows\system32\Dwm.exe"
************************************************************************
explorer.exe pid:   2664
Command line : C:\Windows\Explorer.EXE
************************************************************************
VBoxTray.exe pid:   2792
Command line : "C:\Windows\System32\VBoxTray.exe" 
************************************************************************
cmd.exe pid:   2096
Command line : "C:\Windows\system32\cmd.exe" 
************************************************************************
conhost.exe pid:   2068
Command line : \??\C:\Windows\system32\conhost.exe
************************************************************************
chrome.exe pid:   2296
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" 
************************************************************************
chrome.exe pid:   2304
Command line : 
************************************************************************
chrome.exe pid:   2476
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=watcher --main-thread-id=2312 --on-initialized-event-handle=12 --parent-handle=164 /prefetch:6
************************************************************************
chrome.exe pid:   2964
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=utility --field-trial-handle=920,18321715965689748971,11882971420757355211,131072 --lang=en-US --service-sandbox-type=network --enable-audio-service-sandbox --service-request-channel-token=7874901870054399126 --mojo-platform-channel-handle=1036 /prefetch:8
************************************************************************
chrome.exe pid:   2572
Command line : 
************************************************************************
WmiPrvSE.exe pid:   2636
Command line : C:\Windows\system32\wbem\wmiprvse.exe
************************************************************************
WmiApSrv.exe pid:   2004
Command line : C:\Windows\system32\wbem\WmiApSrv.exe
************************************************************************
chrome.exe pid:   1632
Command line : "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --type=renderer --field-trial-handle=920,18321715965689748971,11882971420757355211,131072 --disable-gpu-compositing --lang=en-US --disable-oor-cors --enable-auto-reload --device-scale-factor=1 --num-raster-threads=1 --service-request-channel-token=14321750869504108513 --renderer-client-id=11 --no-v8-untrusted-code-mitigations --mojo-platform-channel-handle=2096 /prefetch:1
************************************************************************
dllhost.exe pid:   2376
Command line : C:\Windows\system32\DllHost.exe /Processid:{76D0CB12-7604-4048-B83C-1005C7DDC503}
************************************************************************
KeePass.exe pid:   3008
Command line : "C:\Program Files (x86)\KeePass Password Safe 2\KeePass.exe" "C:\Users\SmartNet\Secrets\Hidden.kdbx"
************************************************************************
sppsvc.exe pid:   2764
Command line : C:\Windows\system32\sppsvc.exe
************************************************************************
svchost.exe pid:   1076
Command line : C:\Windows\System32\svchost.exe -k secsvcs
************************************************************************
wmpnetwk.exe pid:    928
Command line : "C:\Program Files\Windows Media Player\wmpnetwk.exe"
************************************************************************
notepad.exe pid:   3260
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\SmartNet\Secrets\Hidden.kdbx
************************************************************************
DumpIt.exe pid:   3844
Command line : "C:\Users\SmartNet\Downloads\DumpIt\DumpIt.exe" 
************************************************************************
conhost.exe pid:   3852
Command line : \??\C:\Windows\system32\conhost.exe
************************************************************************
WmiPrvSE.exe pid:   4004

```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img5.png)

Now i’ll use the following command to find `Hidden.kdbx` locaion in memory dump.

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep Hidden.kdbx
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img6.png)

This gave me the address where this file exists on that memory dump. If we focus on the results of `cmdline`  plugin, someone tried to open that file on notepad, but as it is protected, so unable to do it.So how am I gonna read this one? Well, i can if it’s on my local machine. And you get it, let’s dump it on my machine

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 dumpfiles -Q 000000003fb112a0 -D .
# -Q flag is for address & -D is for the path of diretory where i want to dump the file.
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img7.png)

Here i got it on my machine.

```bash
└─$ ls
MemLabs-Lab2.7z  MemoryDump_Lab2.raw  file.None.0xfffffa8001593ba0.dat
```

I used the `file` command to check it’s format. it is a `.kdbx` file

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img8.png)

let’s rename this file for convenience

```bash
 mv file.None.0xfffffa8001593ba0.dat hidden.kdbx
```

Now i used chatgpt to ask, how to read this file and it told me to use `keepassxc` . I opened the file

```bash
keepassxc hidden.kdbx
```

But you know what? it gave me a prompt to enter password(and i don’t have any). So clearly, a dead end for now. 

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img9.png)

Let’s still get going. The way I found `hidden.kdbx` file, let’s check can we found anything related to passwords

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep -i password
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img12.png)

from all what stoods out is `Password.png` file. Let’s dump it on my machine.

```bash
volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fce1c70 -D .
```

I have renamed the file to file.png. 

 ![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img13.png)

And opened it.

```bash
eog file.png
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img14.png)

This is the image, and if u focus on the corner there is written some password.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img15.png)

I tried this password and here i got it.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img16.png)

I checked all the folders and in the recycle bin , i got the second flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img17.png)

### Flag 2:

```bash
flag{w0w_th1s_1s_Th3_SeC0nD_ST4g3_!!}
```

For the third flag comes the `browsers` hint. In the keepass general folder there are given some username and passwords. Let’s figure where this road gets us(obv to 3rd flag hehe).

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img18.png)

In the processes lists, chrome was running. Let’s dump it’s history and look what i get.

```bash
volatility --plugins=../../volatility-plugins -f MemoryDump_Lab2.raw --profile=Win7SP1x64 chromehistory
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img19.png)

From the history that looks a bit odd.I checked the link, it was MEGA cloud. I downloaded the .zip folder

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img20.png)

I unzipped it, but there was another zip folder in it, that was password protected. But when i simply tried  to unzip it it gave the hint passord is sha1(stage 3 flag of lab 1). So i provided the password and the zipped folder contain an image. When i opened the image it gave me the third flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img21.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img22.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab2/img23.png)

That’s all for today. See you in the next lab.

Happy hunting!!!