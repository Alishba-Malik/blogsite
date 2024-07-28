---

title: MemLab Series-Lab 3
date: '2024-07-28'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Recover encrypted information using steghide from memory dump.

---
# **MemLabs Lab 3 - The Evil's Den**

Welcome back, cyber sleuths, to another thrilling installment of the MemLab Series! Today, we dive headfirst into the enigma that is **Lab 3**. If you've been with me through the labs, you already know the drill: each memory dump is a treasure trove of secrets waiting to be unearthed, and Lab 3 is no exception.

## **Challenge Description:**

A malicious script encrypted a very secret piece of information I had on my system. Can you recover the information for me please?

**Note-1**: This challenge is composed of only 1 flag. The flag split into 2 parts.

**Note-2**: You'll need the first half of the flag to get the second.

You will need this additional tool to solve the challenge,

```
$ sudo apt install steghide
```

The flag format for this lab is: `**inctf{s0me_l33t_Str1ng}**`

**Challenge file**: [MemLabs_Lab3](https://mega.nz/#!2ohlTAzL!1T5iGzhUWdn88zS1yrDJA06yUouZxC-VstzXFSRuzVg)

Let’s fire up the terminal and dive into this lab. After the first three labs, this time I have already unzipped the file and started doing the basic enumeration.

```bash
volatility -f MemoryDump_Lab3.raw imageinfo
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/1.png)

Running the scan on the first profile.

```bash
volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 kdbgscan
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/2.png)

now let’s move on to check the active processes.

```bash
volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 pslist
```

```bash
└─$ volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 pslist
Volatility Foundation Volatility Framework 2.6
Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0x83d09c60 System                    4      0     88      541 ------      0 2018-09-30 08:09:59 UTC+0000                                 
0x84551b98 smss.exe                260      4      2       29 ------      0 2018-09-30 08:09:59 UTC+0000                                 
0x84d58030 csrss.exe               340    332      9      352      0      0 2018-09-30 08:10:04 UTC+0000                                 
0x84d76030 csrss.exe               380    372     10      189      1      0 2018-09-30 08:10:05 UTC+0000                                 
0x84d77d28 wininit.exe             388    332      3       83      0      0 2018-09-30 08:10:05 UTC+0000                                 
0x84da6d28 winlogon.exe            424    372      3      115      1      0 2018-09-30 08:10:05 UTC+0000                                 
0x84dcdbd0 services.exe            484    388      6      195      0      0 2018-09-30 08:10:07 UTC+0000                                 
0x84dd0658 lsass.exe               492    388      6      561      0      0 2018-09-30 08:10:08 UTC+0000                                 
0x84dd4b28 lsm.exe                 500    388     10      151      0      0 2018-09-30 08:10:08 UTC+0000                                 
0x8454e348 svchost.exe             588    484     10      351      0      0 2018-09-30 08:10:12 UTC+0000                                 
0x84e15d28 VBoxService.ex          648    484     12      115      0      0 2018-09-30 08:10:13 UTC+0000                                 
0x84e1d030 svchost.exe             712    484      8      268      0      0 2018-09-30 08:10:14 UTC+0000                                 
0x84e5ad28 svchost.exe             800    484     18      438      0      0 2018-09-30 08:10:14 UTC+0000                                 
0x84e67d28 svchost.exe             852    484     16      371      0      0 2018-09-30 08:10:15 UTC+0000                                 
0x84e6b030 svchost.exe             880    484     18      452      0      0 2018-09-30 08:10:15 UTC+0000                                 
0x84e6fa18 svchost.exe             904    484     31     1116      0      0 2018-09-30 08:10:15 UTC+0000                                 
0x8481bcb0 svchost.exe            1236    484     15      478      0      0 2018-09-30 08:10:22 UTC+0000                                 
0x8484a800 spoolsv.exe            1340    484     12      285      0      0 2018-09-30 08:10:24 UTC+0000                                 
0x8485b030 svchost.exe            1368    484     18      302      0      0 2018-09-30 08:10:24 UTC+0000                                 
0x8488e860 svchost.exe            1488    484     11      267      0      0 2018-09-30 08:10:26 UTC+0000                                 
0x84893030 svchost.exe            1516    484     12      215      0      0 2018-09-30 08:10:26 UTC+0000                                 
0x85192030 LogonUI.exe             876    388      5      152      0      0 2018-09-30 08:10:40 UTC+0000                                 
0x8515cae0 sppsvc.exe              292    484      6      153      0      0 2018-09-30 08:12:31 UTC+0000                                 
0x8514bbf0 svchost.exe             440    484     13      342      0      0 2018-09-30 08:12:32 UTC+0000                                 
0x84d69d00 SearchIndexer.         1184    484     15      724      0      0 2018-09-30 08:12:33 UTC+0000                                 
0x8441d7e0 taskhost.exe           4816    484      8      196      1      0 2018-09-30 09:28:32 UTC+0000                                 
0xa0b21170 dwm.exe                3028    852      3      186      1      0 2018-09-30 09:28:36 UTC+0000                                 
0x8449d890 explorer.exe           5300   5128     30      871      1      0 2018-09-30 09:28:36 UTC+0000                                 
0x851cdd28 VBoxTray.exe           3064   5300     14      154      1      0 2018-09-30 09:28:44 UTC+0000                                 
0x84d77868 wuauclt.exe            5644    904      3       86      1      0 2018-09-30 09:28:49 UTC+0000                                 
0x9c627d28 msiexec.exe            1016    484      7      345      0      0 2018-09-30 09:39:03 UTC+0000                                 
0xbc2d08a8 msiexec.exe            5652   1016      0 --------      1      0 2018-09-30 09:39:13 UTC+0000   2018-09-30 09:41:17 UTC+0000  
0xbc21b9f0 TrustedInstall         4724    484      4      139      0      0 2018-09-30 09:40:24 UTC+0000                                 
0x84489800 audiodg.exe            5996    800      4      120      0      0 2018-09-30 09:45:22 UTC+0000                                 
0x83fbba40 SearchProtocol         5748   1184      7      281      0      0 2018-09-30 09:45:32 UTC+0000                                 
0x84ead628 DumpIt.exe             4116   5300      2       37      1      0 2018-09-30 09:45:43 UTC+0000                                 
0x84e37498 conhost.exe            3176    380      2       51      1      0 2018-09-30 09:45:43 UTC+0000                                 
0x84700ab8 dllhost.exe            1008    588      8      225      1      0 2018-09-30 09:45:48 UTC+0000                                 
0x84ef6768 SearchFilterHo         4036   1184      5       97      0      0 2018-09-30 09:47:36 UTC+0000                                 
0x9c6b0970 notepad.exe            3736   5300      1       60      1      0 2018-09-30 09:47:49 UTC+0000                                 
0x8443d3c0 notepad.exe            3432   5300      1       60      1      0 2018-09-30 09:47:50 UTC+0000                                 

```

From all the active processes, `notepad.exe` seems suspicious, as it’s also mentioned in the challenge description that some malicious script encrypted a very secret piece of information on the system. So I checked what arguments were passed with notepad.exe.

```bash
volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 cmdline 
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/3.png)

I dumped both files on my machine to give them a quick read.First i dumped `vip.txt`.

```bash
volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 filescan | grep vip.txt

volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000003e727e50 -D .
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/4.png)

Then I changed its name for convenience.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/5.png)

Then dumped the other one.

```bash
 volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 filescan | grep evilscript.py 
```

```bash
volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000003de1b5f0 -D .
```

```bash
file file.None.0xbc2b6af0.dat
mv file.None.0xbc2b6af0.dat evilscript.py
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/6.png)

I read vip.txt. It had a base64 string. I decoded it, but it gave some gibberish output.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/7.png)

This was the script in evilscript.py.

```bash
import sys
import string

def xor(s):

	a = ''.join(chr(ord(i)^3) for i in s)
	return a

def encoder(x):
	
	return x.encode("base64")

if __name__ == "__main__":

	f = open("C:\\Users\\hello\\Desktop\\vip.txt", "w")

	arr = sys.argv[1]

	arr = encoder(xor(arr))

	f.write(arr)

	f.close()
```

In this script, whatever input is provided as an argument will first be XORed. Then it’ll be encoded in base64 and stored in vip.txt. We have the base64 string already, so why not reverse the process? I wrote a basic decoding script for this:

```bash
import base64
import sys

def xor(s):
    return ''.join(chr(ord(i) ^ 3) for i in s)

def decoder(x):
    return base64.b64decode(x).decode('utf-8')

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python decoder.py <arr>")
        sys.exit(1)

    arr = sys.argv[1]

    flag = xor(decoder(arr))
    print("Decoded string:", flag)

```

As I ran this script and provided that base64 string as an argument, it printed the first half of the flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/8.png)

Now the drill is for the second half. And the hint for this one is steghide. So I opened the steghide man page, and this is what I found.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/9.png)

Steghide is limited to certain files. Let’s `filescan` for these file, I’ll start with jpeg.

```bash
 volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 filescan | grep jpeg
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/10.png)

i found a suspicion1.jpeg file. Let’s dump it.

```bash
 volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 dumpfiles -Q 0x0000000004f34148 -D .
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/11.png)

I renamed this file to pic.jpeg. Ran the command 

```bash
steghide extract -sf pic.jpeg 
```

I entered the first half of the flag as the passphrase (as it was mentioned in the description that you need the first half of the flag for the next half). It extracted a “secret text” file. I read it and got the second half of the flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab3/12.png)

### Flag:

```bash
inctf{0n3_h4lf_1s_n0t_3n0ugh}
```
That's it for today. **Happy Hunting.**