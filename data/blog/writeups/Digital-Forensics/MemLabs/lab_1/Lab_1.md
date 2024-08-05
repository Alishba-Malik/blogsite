---

title: MemLab Series-Lab 1
date: '2024-07-26'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Recover important files from crashed system's memory dump.

---
# **Lab 1 - Beginner's Luck**

Hey there, fellow cyber enthusiasts! Buckle up, because today, we're diving into another lab.

### **Challenge description**

My sister's computer crashed. We were very fortunate to recover this memory dump. Your job is get all her important files from the system. From what we remember, we suddenly saw a black window pop up with some thing being executed. When the crash happened, she was trying to draw something. Thats all we remember from the time of crash.

**Challenge file**: [MemLabs_Lab1](https://mega.nz/#!6l4BhKIb!l8ATZoliB_ULlvlkESwkPiXAETJEF7p91Gf9CWuQI70)

Sounds like a plot from a tech-thriller, right? Let's unravel this together!

This challenge is packed with three flags to uncover. Armed with the memory dump, we set out on our forensics adventure. Here’s how the saga unfolds:

First things first, let’s unzip the lab file. Use the command below to extract the contents:

```bash
7z x MemLabs-Lab1.7z
```

With the files ready, it’s time to probe into the memory dump using Volatility.To analyze the memory dump, I need to identify the correct profile. Here’s how I did it:

```bash
volatility -f MemoryDump_Lab1.raw imageinfo
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/1.png)

Next, to ensure we have the right profile, I  ran:

```bash
volatility -f MemoryDump_Lab1.raw kdbgscan 
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/2.0.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/2.1.png)

With the profile confirmed as Win7SP1x64, let’s list the active processes:

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/3.png)

From the description,the biggest hints for me were drawing,something being executed and important files. So the saga starts from this point.FRom the process list what captures my attention were

- cmd.exe
- mspaint.exe
- WinRAR.exe

Given the black window hint, we dive into the command history:

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 cmdscan
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/4.png)

Among the findings, this string caught my eye:

```bash
St4G3$1
```
Hmm, intriguing! Let's dig deeper by examining the console history:

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 consoles
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/5.png)

A base64 encoded string surfaced in my search:

```bash
ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0= 
```

Decoding it revealed our first flag:

```bash
echo -n "ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0=" | base64 -d
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/6.png)
### flag 1:

```bash
flag{th1s_1s_th3_1st_st4g3!!}
```

1 hint down to more to go. So for this i started digging on mspaint.exe, as it’s pid was mentioned in the process list. I memdumped the paint to get to know what was being drawn at that time.

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 2424 -D .
#-p for process id
# -D for directory where u wanna dump the file.
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/7.png)

Renaming the dumped file for clarity:

```bash
mv 2424.dmp 2424.data
```

Using GIMP to visualize the data . Offset and dimensions courtesy of The Cyber Expert coz it was way guessy and gonna took years (at least for me; otherwise my procratination would mind hehe) to figure out.

```bash
gimp 2424.data 
```

With offset 6774541, width 1230, and height 10000, I extracted the second flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/8.0.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/8.1.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/8.2.png)

### Flag 2:

```bash
flag{good_boy_bad_girl}
```

Finally, I turn my attention to the command-line arguments of processes to locate a suspicious RAR file (the last hint):

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 cmdline
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/9.png)

As our current target is WinRAR file, so as the argument passed with it 

```bash
C:\Users\Alissa Simpson\Documents\Important.rar
```

now let’s scan this file to get it’s address.

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 filescan | grep -i important.rar
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/10.png)

Dumping the file:

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fa3ebc0 -D .
ls
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/11.png)

Renaming and extracting the RAR:

```bash
file file.None.0xfffffa8001034450.dat
mv file.None.0xfffffa8001034450.dat important.rar
unrar -e important.rar
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/12.png)

The important.rar file was password protected. But when i ran the `unrar` command, it gave the hint for password, which was NTLM hash of alisa’s account. let’s hashdump it. 

```bash
volatility -f MemoryDump_Lab1.raw --profile=Win7SP1x64 hashdump
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/13.png)

password: F4FF64C8BAAC57D22F22EDC681055BA6

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/14.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/15.png)

```bash
eog flag3.png
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab1/16.png)

### flag 3:

```bash
flag{w3ll_3rd_stage_was_easy}
```

And there you have it! Through a series of clever forensics steps, we’ve uncovered all three flags hidden within the memory dump. **Stay curious, keep exploring, and happy hunting!**