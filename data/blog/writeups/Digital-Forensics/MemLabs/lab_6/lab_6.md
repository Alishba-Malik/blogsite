---

title: MemLab Series-Lab 6
date: '2024-08-05'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Recover the data from the chrome history of a memory dump.

---

# **MemLabs Lab 6 - The Reckoning**

Hey memory forensics aficionados! We're at the final stretch of our memory forensics journey with MemLabs Lab 6. It's been a wild ride through the intricate maze of RAM, uncovering secrets and piecing together puzzles.  This isn't just another challenge; it's the ultimate test of everything we've learned so far. Buckle up, because we're about to dive deep into the intricate world  where every byte has a tale to tell, and every clue inches us closer to uncovering the truth.  Let’s see what hidden gems we can discover this last time!

### **Challenge Description**

We received this memory dump from the Intelligence Bureau Department. They say this evidence might hold some secrets of the underworld gangster David Benjamin. This memory dump was taken from one of his workers whom the FBI busted earlier this week. Your job is to go through the memory dump and see if you can figure something out. FBI also says that David communicated with his workers via the internet so that might be a good place to start.

**Note**: This challenge is composed of 1 flag split into 2 parts.

The flag format for this lab is: ```inctf{s0me_l33t_Str1ng}```

**Challenge file**: [MemLabs_Lab6](https://mega.nz/#!C0pjUKxI!LnedePAfsJvFgD-Uaa4-f1Tu0kl5bFDzW6Mn2Ng6pnM)

I know it’s the last lab, but we'll start with unzipping the zipped file. HAHAHA

```bash
7z x MemLabs-Lab6.7z
```

Then I’ll proceed to get the image info and run a quick kdbgscan to check the credibility of the profile.

```bash
volatility -f MemoryDump_Lab6.raw imageinfo
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/1.png)

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 kdbgscan
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/2.png)

Moving on to listing all the processes and sorting out the suspicious ones.

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 pslist
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/3.png)

As mentioned in the challenge description, the internet might be a good place to start. Though in the process list there is a .rar folder, we'll take a look at it later. What strikes me as odd is chrome.exe and morzilla.exe. So, let’s start by dumping the chromehistory plugin. But before that, let’s take a look at the command line.

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 cmdline
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/4.png)

What I found interesting was the flag.rar file. But I’ll first go with the internet. I used this command to get an overview of Chrome history.

```bash
volatility --plugins=../../volatility-plugins -f  MemoryDump_Lab6.raw --profile=Win7SP1x64 chromehistory
```

I found a lot of junky stuff and many links. I went through all the possible links, but one that could lead us to the flag is the pastebin link. So, I checked it out and found a document. I went through the document and found an encrypted drive.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/5.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/6.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/7.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/8.png)

Maybe .winrar has that decryption key. So let’s filescan that flag.rar and dump it on my machine.

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 filescan | grep -i "flag.rar"

```

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000005fcfc4b0 -D .
file file.None.0xfffffa800138d750.dat 
mv file.None.0xfffffa800138d750.dat flag.rar
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/9.png)

I unzipped it but had no luck as it was password protected as well.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/10.png)

Now, to look for the password, let’s take a look at the clipboard and screenshots. Maybe we got something from there. I didn’t find anything on the clipboard, but yeah, I got something from the screenshots.

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 clipboard 
```

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 screenshot -D .

```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/11.png)

I checked all the screenshots, and one stood out.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/12.png)

So, I ran a string command on the memory dump and grepped the Mega Drive key. From all the rubbish stuff, what to focus on was a decryption key.

```bash
strings MemoryDump_Lab6.raw | grep -i "Mega Drive Key"
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/13.png)

I entered the key and found a flag.png file. I downloaded it and opened it, but it said IHDR missing. I looked over the internet about the PNG IHDR header and found this image. I opened the downloaded image in hex and compared the hex values. There was a flaw in one of the bytes of the IHDR header, so I corrected it and opened the image. Wallah! I got the first part of the flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/14.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/15.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/16.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/17.png)

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/18.png)

Now the drill is to look for that flag.rar password.  The only option we are left with is to check environment variables. Let’s dig into it.

```bash
volatility -f MemoryDump_Lab6.raw --profile=Win7SP1x64 envars
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/19.png)

I got the password. i entered it and extracted the flag.png. Here we go, the second part of the flag.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab6/20.png)

### Flag:

```bash
inctf{thi5_cH4LL3Ng3_!s_g0nn4_b3_aN_Am4zINg_!_i_gU3Ss???_}
```

And there you have it, folks! This journey through MemLabs has been enlightening, pushing us to think critically and sharpening our forensic skills. Whether you’re a seasoned expert or a curious beginner, I hope you’ve enjoyed this series as much as I have. Stay curious, keep exploring, and remember: **Every byte has a story to tell**.

**Happy hunting!**