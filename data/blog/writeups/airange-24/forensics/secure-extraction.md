---
title: AIRANGE'24 - Forensics - Secure Extraction
date: '2024-03-10'
tags: ['ctf', 'forensics', 'airange', 'writeup', 'john', 'binwalk']
draft: false
summary: An engaging and straightforward forensics challenge that incorporates the utilization of renowned tools such as John the Ripper and binwalk.
---

## Challenge Description

I swear it was in the file.

[Download](https://airange.online/files/269f0e295127d6f11de9c5fea8a08f46/chall.zip?token=eyJ1c2VyX2lkIjo3MSwidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6MjZ9.Ze2IlA.Fp5tuOBkL4jZfy3BcaUCCGMsYlQ)


## Solution

Provided zip file is password protected:

![Screenshot](/static/writeups/airange24/forensics/secure_extraction_1.png)

I used a program called `**John the Ripper**` to figure out the password. First, I used another tool called **`zip2john`** to grab the password codes from the zip file and store them in a file called hash.txt.:

```perl
zip2john chall.zip > hash.txt
```

Command for john:

```python
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Password is **`easypass`**:

![Screenshot](/static/writeups/airange24/forensics/secure_extraction_2.png)

Now after extracting file using this password, I got this file:

![Screenshot](/static/writeups/airange24/forensics/secure_extraction_3.png)

I was thinking about a way to open docx file in linux but then I thought why not use **`binwalk`**?

Command:

```python
binwalk -e challenge.docx
```

We got a file named **`flag.png`**:

![Screenshot](/static/writeups/airange24/forensics/secure_extraction_4.png)

Here is the flag:

![Screenshot](/static/writeups/airange24/forensics/secure_extraction_5.png)

**`Flag.png`**:

![Screenshot](/static/writeups/airange24/forensics/secure_extraction_6.png)

Flag:

```
AUCSS{z1Pp3R_g1V3s_u_w1nG$}
```