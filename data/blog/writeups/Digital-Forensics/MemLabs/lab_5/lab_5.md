---

title: MemLab Series-Lab 5
date: '2024-07-31'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Investigate strange files and a crashing application from a memory dump.

---
# **MemLabs Lab 5 - Black Tuesday**

Welcome back, cyber sleuths! Ready to dive into another thrilling memory forensics adventure? This time, we’re tackling MemLabs Lab 5 - Black Tuesday. Our client has handed over a memory dump with some intriguing and suspicious activity. Let’s roll up our sleeves, fire up our forensic tools, and get to the bottom of this digital mystery.

### **Challenge Description**

We received this memory dump from our client recently. Someone accessed his system when he was not there and he found some rather strange files being accessed. Find those files and they might be useful. I quote his exact statement,

> The names were not readable. They were composed of alphabets and numbers but I wasn't able to make out what exactly it was.
> 

Also, he noticed his most loved application that he always used crashed every time he ran it. Was it a virus?

**Note-1**: This challenge is composed of 3 flags. If you think 2nd flag is the end, it isn't!! :P

**Note-2**: There was a small mistake when making this challenge. If you find any string which has the string `L4B_3_D0n3*!!` in it, please change it to `L4B_5_D0n3*!!` and then proceed.

**Note-3**: You'll get the stage 2 flag only when you have the stage 1 flag.

**Challenge file**: [MemLabs_Lab5](https://mega.nz/#!Ps5ViIqZ!UQtKmUuKUcqqtt6elP_9OJtnAbpwwMD7lVKN1iWGoec)

Like always, unzip the file and run `imageinfo` command on the memory dump

```bash
7z x MemLabs-Lab5.7z
```

```bash
volatility -f MemoryDump_Lab5.raw imageinfo 
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/1.png)

Now, let’s run `kdbgscan` as usual to check the credibility of the profile we'll be working on.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 kdbgscan
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/2.png)

From the challenge hints, we know:

- Strange files have been accessed
- The file names are not readable
- An application always crashes when opened
- You need the stage 1 flag for stage 2

Let's run the `pslist` command to identify any suspicious processes.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 pslist
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/3.png)

The files that appear suspicious to me are `notepad.exe`, which I think is the file that crashes since it has been opened multiple times. Let's check the command-line arguments for these processes using the `cmdline` command.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 cmdline 
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/4.png)

If we closely look at `NOTEPAD.EXE`, it’s not run from `system32`. **Feels like a virus, right? At least to me.** First, let's investigate the `.rar` file. We can find its address using the `filescan` plugin.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 filescan | grep -i "SW1wb3J0YW50.rar"

```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/5.png)

Dumping the file on my machine.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003eed56f0 -D . 

```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/6.png)

Rename the file for convenience and `unrar` it.

```bash
file file.None.0xfffffa80010b44f0.dat  
mv file.None.0xfffffa80010b44f0.dat file.rar 
unrar e file.rar 
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/7.png)

**Wait, what?** The file is password protected, and the password is the stage 1 flag. We can't proceed further. Let’s pause this process here and figure out how to get the stage 1 flag. Like in previous blogs, I ran the `clipboard` plugin in hopes of finding something interesting.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 clipboard
```
No luck there. So, I moved on to screenshots.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 screenshot -D .
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/8.png)

Dumped all the screenshots, now let’s take a look at them.

```bash
eog session*
strings MemoryDump_Lab5.raw | grep "Zmxh"
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/9.png)

I found a base64 string. Let's decode it; it could be a clue.

```bash
echo -n "ZmxhZ3shIV93M0xMX2QwbjNfU3Q0ZzMtMV8wZl9MNEJfNV9EMG4zXyEhfQ" | base64 -d
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/10.png)

wallah, we got our first flag and password of the .rar file.

### Flag:
```bash
 flag{!!*w3LL_d0n3_St4g3-1_0f_L4B_5_D0n3*!!}
 ```

let’s now unrar the file. I got a .png file, opened it got another flag. **HURRAY!!!**

```bash
unrar e file.rar
eog Stage2.png

```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/11.png)

Now we are left with the virus. First, get its address and dump it on my machine for reverse engineering.

```bash
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 filescan | grep -i "notepad.exe"
volatility -f MemoryDump_Lab5.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003ee9d070 -D .
file file.None.0xfffffa8000f886e0.dat
mv file.None.0xfffffa8000f886e0.dat file.exe
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/12.png)

I did the usual stuff and opened it in IDA. Whoa, got my flag.

```bash
ida64 file.exe
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/13.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab5/14.png)


### final flag:

```bash
bi0s{M3m_14B5_OVeR_!}
```

And there we have it! Another challenge cracked wide open. Until next time, **keep those bytes in check and happy hunting!**