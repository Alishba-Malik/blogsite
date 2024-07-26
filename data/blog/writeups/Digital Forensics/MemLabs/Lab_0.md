---

title: MemLab Series-Lab 0
date: '2024-07-26'
tags: ['Digital Forensics', 'ctf', 'Memory dump', 'Volatility','MemLabs']
draft: false
summary: Given memory dump, find user activity and flag.A sample challenge.

---


# **Lab 0 - Never Too Late Mister**
Welcome to the captivating world of digital forensics, where every byte is a clue and every memory dump holds the key to uncovering hidden truths. As I embark on my journey through the MemLabs series, Lab 0 emerges as the perfect starting point—a blend of simplicity and intrigue that lays the groundwork for more complex challenges ahead. In this foundational lab, we delve into the essentials of memory analysis, wielding fundamental tools and techniques that are crucial for any digital detective. Join me as we decode the mysteries hidden within volatile memory, uncovering the secrets that lie just beneath the surface in our first step into the enthralling domain of digital forensics.

Lab 0 is a sample challenge whose description is also provided, setting the stage for the upcoming ones. Without any delay, let’s dig into it.

Let’s start with the challenge description. But first, let me provide you with the [challenge link](https://github.com/stuxnet999/MemLabs/tree/master).

[Challenge file](https://drive.google.com/file/d/1MjMGRiPzweCOdikO3DTaVfbdBK5kyynT/view)

The challenge description says:

**My friend John is an "environmental" activist and a humanitarian. He hated the ideology of Thanos from the Avengers: Infinity War. He sucks at programming. He used too many variables while writing any program. One day, John gave me a memory dump and asked me to find out what he was doing while he took the dump. Can you figure it out for me?**

First things first, let’s unarchive the tar file.

```bash
tar -xvf Challenge.tar.xz
```

Now, let’s play with the dump file. I’m going to use Volatility 2.6 on my Linux machine because these challenges are a bit old, so the older version of Volatility will work best for them. To see info about the image, I’ll use the command:

```bash
volatility -f Challenge.raw imageinfo
```
![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img1.png)
The result shows three profiles are running:

- Win7SP1x86_23418
- Win7SP0x86
- Win7SP1x86

You can choose any of these profiles, but I’ll choose Win7SP1x86 for the memory forensics. Nothing special, it’s just my choice (hehe). However, most of the time, all the profiles listed may not work. Then this command steps in as a savior:

```bash
volatility -f Challenge.raw kdbgscan
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img2.png)

For this challenge, if I run this command, nothing special shows up, so all the profiles are feasible to work on. Moreover, all the plugins I’m using can be checked [here](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference). 

As a analyst i’ll first work on these things

- Chect the active and hidden processes
- Check the recent commands executed
- Check the history

So the first thing i’ll do is list all the active processes working on this image using the command:

```bash
volatility -f Challenge.raw --profile=Win7SP1x86 pslist
#pslist -> is to list all currently running processses
```

It’ll take some time to list all the processes. Here are the results.  

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img3.png)

After having a closer look at these processes, the one that seems a bit odd is cmd.exe. This is because it’s not one of the processes that works in the background. So let’s dig a bit by running a scan on cmd using the plugin cmdscan.

```bash
volatility -f Challenge.raw --profile=Win7SP1x86 cmdscan
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img4.png)

From the results, one that stands out is 

```bash
Cmd #0 @ 0x2f43c0: C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
```

To check the output of this command, I’ll use the consoles plugin, so it shows the output as stdout.

```bash
volatility -f Challenge.raw --profile=Win7SP1x86 consoles
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img5.png)

The interesting thing I got is this hex-looking number: 335d366f5d6031767631707f. On my terminal, I used Python to unhex it, but it gave a kind of gibberish text.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img6.png)

```bash
b'3]6o]`1vv1p\x7f'
```

This seems like a dead end. What about jumping back to challenge description for some hope?

Wallah, we got some. It has quoted “environmental” and then mentioned many variables. This points to environmental variables. Let’s use the envars plugin for this and see what we get.

```bash
volatility -f Challenge.raw --profile=Win7SP1x86 envars
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img7.png)

After seeing the results of environment variables and considering the challenge description, I got a hint of thanos. As I have highlighted, it tells, xor and password. Let’s xor that hex string (I’m gonna brute force, as I don’t have any other option left). For this, I’ll use Cyberchef.

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img8.png)

Seems like the output of key 2 is the 2nd half part of flag

```bash
Key = 02: 1_4m_b3tt3r}
```

Now, the quest is for the first part. Here, if we again focus on the Thanos hint, other than xor there was a hint of password. As passwords are stored in SSD not in RAM, when the user enters the passwords, their hashes are stored in cache to remember for the future. So let’s dump the cache hashes:

```bash
volatility -f Challenge.raw --profile=Win7SP1x86 hashdump
```

![Screenshot](/static/writeups/Digital_forensics/Memlabs/lab0/img9.png)

Here we go, we got some NTLM hashes. If we closely look out of three, two are the default users i.e., Administrator and Guest, but the third one hello is not a default one. Let’s crack its hash. For this, I tried CrackStation, John, and multiple online websites, but didn’t get the answer. So I checked the issues on GitHub and it mentioned that the website used for this hash had renewed the database. So now, I’m using the last straw mimikatz plugin:

```bash
volatility -f Challenge.raw --profile=Win7SP1x86 hashdump --plugins=/usr/bin/volatility-plugins/plugins mimkatz
```

Still unable to get the first part of the flag. As the issue was genuine, the database has been renewed. That’s why in one of the issues they have mentioned that they had provided the walkthrough for this one.

so the flag is:

```bash
flag{you_are_good_but1_4m_b3tt3r}
```
**Happy Hacking!!!**