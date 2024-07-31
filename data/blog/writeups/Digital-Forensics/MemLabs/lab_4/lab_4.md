---

title: MemLab Series-Lab 4
date: '2024-07-30'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Recover deleted file from memory dump using volatility.

---

# **MemLabs Lab 4 - Obsession**

In this lab of the Memlab Series, we're diving into a real-life nightmare scenario for any system administrator. Imagine this: your system has been breached by a hacker who not only steals sensitive information but also deletes an invaluable file. The only clue left behind? A memory dump.  It's a race against time to piece together the puzzle and recover the lost data while uncovering the hacker's trail.

### **Challenge Description**

My system was recently compromised. The Hacker stole a lot of information but he also deleted a very important file of mine. I have no idea on how to recover it. The only evidence we have, at this point of time is this memory dump. Please help me.

**Note**: This challenge is composed of only 1 flag.

The flag format for this lab is: `inctf{s0me_l33t_Str1ng}`

**Challenge file**: [MemLabs_Lab4](https://mega.nz/#!Tx41jC5K!ifdu9DUair0sHncj5QWImJovfxixcAY-gt72mCXmYrE)

Let's fire up the terminal by unzipping the challenge file and running an imageinfo on the dump file.

```bash
7z x MemLabs-Lab4.7z 
```

```bash
 volatility -f MemoryDump_Lab4.raw imageinfo 
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/1.png)

First, we'll check if the profile is correct to work on by running kdbgscan. Then we proceed to enumerate the processes.

```bash
                                                                                
┌──(alishba㉿kali)-[~/…/cyber_dir/forensics/memlabs/lab_4]
└─$ volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 kdbgscan
Volatility Foundation Volatility Framework 2.6
**************************************************
Instantiating KDBG using: Kernel AS Win7SP1x64 (6.1.7601 64bit)
Offset (V)                    : 0xf800027f60a0
Offset (P)                    : 0x27f60a0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP1x64
Version64                     : 0xf800027f6068 (Major: 15, Minor: 7601)
Service Pack (CmNtCSDVersion) : 1
Build string (NtBuildLab)     : 7601.17514.amd64fre.win7sp1_rtm.
PsActiveProcessHead           : 0xfffff8000282cb90 (41 processes)
PsLoadedModuleList            : 0xfffff8000284ae90 (138 modules)
KernelBase                    : 0xfffff80002605000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 1
KPCR                          : 0xfffff800027f7d00 (CPU 0)

**************************************************
Instantiating KDBG using: Kernel AS Win7SP1x64 (6.1.7601 64bit)
Offset (V)                    : 0xf800027f60a0
Offset (P)                    : 0x27f60a0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win2008R2SP1x64
Version64                     : 0xf800027f6068 (Major: 15, Minor: 7601)
Service Pack (CmNtCSDVersion) : 1
Build string (NtBuildLab)     : 7601.17514.amd64fre.win7sp1_rtm.
PsActiveProcessHead           : 0xfffff8000282cb90 (41 processes)
PsLoadedModuleList            : 0xfffff8000284ae90 (138 modules)
KernelBase                    : 0xfffff80002605000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 1
KPCR                          : 0xfffff800027f7d00 (CPU 0)

**************************************************
Instantiating KDBG using: Kernel AS Win7SP1x64 (6.1.7601 64bit)
Offset (V)                    : 0xf800027f60a0
Offset (P)                    : 0x27f60a0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP1x64_23418
Version64                     : 0xf800027f6068 (Major: 15, Minor: 7601)
Service Pack (CmNtCSDVersion) : 1
Build string (NtBuildLab)     : 7601.17514.amd64fre.win7sp1_rtm.
PsActiveProcessHead           : 0xfffff8000282cb90 (41 processes)
PsLoadedModuleList            : 0xfffff8000284ae90 (138 modules)
KernelBase                    : 0xfffff80002605000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 1
KPCR                          : 0xfffff800027f7d00 (CPU 0)

**************************************************
Instantiating KDBG using: Kernel AS Win7SP1x64 (6.1.7601 64bit)
Offset (V)                    : 0xf800027f60a0
Offset (P)                    : 0x27f60a0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win2008R2SP0x64
Version64                     : 0xf800027f6068 (Major: 15, Minor: 7601)
Service Pack (CmNtCSDVersion) : 1
Build string (NtBuildLab)     : 7601.17514.amd64fre.win7sp1_rtm.
PsActiveProcessHead           : 0xfffff8000282cb90 (41 processes)
PsLoadedModuleList            : 0xfffff8000284ae90 (138 modules)
KernelBase                    : 0xfffff80002605000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 1
KPCR                          : 0xfffff800027f7d00 (CPU 0)

**************************************************
Instantiating KDBG using: Kernel AS Win7SP1x64 (6.1.7601 64bit)
Offset (V)                    : 0xf800027f60a0
Offset (P)                    : 0x27f60a0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP0x64
Version64                     : 0xf800027f6068 (Major: 15, Minor: 7601)
Service Pack (CmNtCSDVersion) : 1
Build string (NtBuildLab)     : 7601.17514.amd64fre.win7sp1_rtm.
PsActiveProcessHead           : 0xfffff8000282cb90 (41 processes)
PsLoadedModuleList            : 0xfffff8000284ae90 (138 modules)
KernelBase                    : 0xfffff80002605000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 1
KPCR                          : 0xfffff800027f7d00 (CPU 0)

**************************************************
Instantiating KDBG using: Kernel AS Win7SP1x64 (6.1.7601 64bit)
Offset (V)                    : 0xf800027f60a0
Offset (P)                    : 0x27f60a0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win2008R2SP1x64_23418
Version64                     : 0xf800027f6068 (Major: 15, Minor: 7601)
Service Pack (CmNtCSDVersion) : 1
Build string (NtBuildLab)     : 7601.17514.amd64fre.win7sp1_rtm.
PsActiveProcessHead           : 0xfffff8000282cb90 (41 processes)
PsLoadedModuleList            : 0xfffff8000284ae90 (138 modules)
KernelBase                    : 0xfffff80002605000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 1
KPCR                          : 0xfffff800027f7d00 (CPU 0)

                                                            
```

Let’s move on to real stuff. And start with running the processes.

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 pslist
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/3.png)

Something seems odd with the `stickynote.exe` process. Sticky Note doesn't typically take arguments, but for our basic enumeration to figure out the deleted file, we'll check the command line inputs, just in case we find something useful. 

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 cmdline
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/2.png)

Nothing particularly interesting there. Next, we'll do a bit of hit and trial. Let's check the clipboard; maybe some data copied there could be the content of the deleted file.

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 clipboard
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/4.png)

No luck with the clipboard. How about screenshots? While screenshots won't give us a clear picture, they might help us see some text or the layout of what was open at that time.

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 screenshot -D .
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/5.png)

Let’s take a look on all screenshots.

```bash
eog session*
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/6.png)

From all the screenshots, one stands out. There's some address in the address bar, which is the biggest clue. It shows something in the `C drive` in the `Users` folder. So, why not run a file scan on Users and be more specific with extensions like jpg, txt, py, jpeg, exe, and png?

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 filescan | grep '\\Users\\' | grep "\jpg\|txt\|py\|jpeg\|exe\|png"
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/7.png)

Among the files listed, three stand out: `galf.jpeg`, `screenshot1.png`, and `important.txt`. Let's dump them on my machine.

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003e8ad250,0x000000003e8d19e0,0x000000003fc398d0 -D .
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/8.png)

We dumped three files, but I only got two files on my machine, which means one of the files was deleted. Kudos to me for figuring that out (hehe). How did I do it? I ran the `file` command on both dumped files. One had a `.png` extension, and the other had a `.jpeg` extension. The one that's missing is our dearly `important.txt` file.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/9.png)

Now, the question remains: how can we retrieve that file? Right? I had the same question (hehe). So I googled (ChatGPT) and innocently asked how we can get the content of a deleted file from a memory dump, especially if we know which file was deleted. It simply said "MFTTables." I then used the `mftparser` plugin to extract MFT entries from the memory dump. This plugin parses the $MFT and can provide details about both active and deleted files. I ran the following command:

```bash
volatility -f MemoryDump_Lab4.raw --profile=Win7SP1x64 mftparser > deleted_file.txt
```

Now let’s read the content of our specific file

```bash
cat deleted_file.txt | grep -i "important.txt" -A 40  
# - A 40 -> to display first 40 lines
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab4/10.png)

Here we go, got our flag.

### Flag:

```bash
inct{1_is_n0t_EQu4l_70_2_but_th1s_d0s3nt_m4ke_s3ns3}
```

That’s it for today. **Keep digging, stay curious, and Happy sleuthing!**