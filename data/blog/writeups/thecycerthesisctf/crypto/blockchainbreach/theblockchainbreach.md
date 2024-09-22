---
title: TheCyberThesis'24 - Cryptography - The Blockchain breach
date: '2024-09-22'
tags: ['Cryptography', 'ctf', 'Blockchain','Writeup', 'MD5 hash collision']
draft: false
summary: Recognize the pattern of a dataset to breach the blockchain.
---

Hey fellawss!!!

I recently had the chance to be a part of *TheCyberThesis CTF*, a 12-hour marathon of brain-melting challenges, and let me tell you—it was wild! I got to create a cryptography challenge that turned out to be one of the hardest I’ve ever designed. And now, I’m here to walk you through it! Buckle up, because this one’s not for the faint of heart (i am just kidding).

So let’s get started!

![Screenshot](/static/writeups/thecyberthesisctf/crypto/blockchainbreach/0.png)



### The Challenge Breakdown

After you dive into this challenge, you’ll be given a website link to load a file named `blockchain.json` given in the zip file. Once you hop onto the site and load the file, you’ll see a dataset with **50 blocks**. Each block contains:

- A **previous hash**
- **Timestamp**
- **Transactions**
- The **block's hash**
- A **salt** used to generate that hash.

![Screenshot](/static/writeups/thecyberthesisctf/crypto/blockchainbreach/1.png)

But let’s get one thing straight: the hashes we’re working with here? Using any online tool we'll get to know they’re MD5! Yep, the old, insecure MD5—the same hash known for its vulnerabilities, like **collisions**. That's your first clue!

![Screenshot](/static/writeups/thecyberthesisctf/crypto/blockchainbreach/2.png)

### Step 2: Cracking the Salt

Moving on to the challenge description:

> Remember, if the salt is shared, unite the fragments of their identity and forge a singular mark from their union.
> 

It hints that some blocks share the same salt, which leads to hash collisions. And guess what? The hint emphasizes that **collisions** are common in weak hashing algorithms like MD5.

But how do we find those blocks with the same salt? Luckily, you also get a file called `salts.json`. Time to whip up a simple script to find which salt is being reused:

```python

import json
from collections import Counter

with open('salts.json') as f:
    salts = json.load(f)

salt_counts = Counter(salts)
most_common_salts = salt_counts.most_common()

for salt, count in most_common_salts:
    print(f"Salt: {salt}, Occurrences: {count}")

```

Running this script gives us the result we’re looking for:

```python

Salt: 5f06ac9dacd3e4169912887a921a49c4, Occurrences: 15

```

### Step 3: Uniting the Fragments

Now that we’ve found the magic salt, the challenge description tells us to "unite the fragments." This means concatenating the hashes of all blocks that share this salt. Then, we hash this concatenated string using MD5 to create a new, singular hash. Here’s the code for that:

```python

import json
import hashlib

def compute_md5_hash(data):
    return hashlib.md5(data.encode()).hexdigest()

def process_blocks(file_path, target_salt):
    with open(file_path, 'r') as file:
        blockchain_data = json.load(file)

    hashes = [block['hash'] for block in blockchain_data if block.get('salt') == target_salt and block.get('hash')]

    if hashes:
        concatenated_hashes = ''.join(hashes)
        return compute_md5_hash(concatenated_hashes)
    else:
        print(f"No blocks found with salt {target_salt}.")
        return None

if __name__ == "__main__":
    new_md5_hash = process_blocks('blockchain.json', '5f06ac9dacd3e4169912887a921a49c4')
    if new_md5_hash:
        print(f"New MD5 Hash: {new_md5_hash}")

```

When we run this script, we get the new MD5 hash:

```python
New MD5 Hash: 0bf207d87de0345bf5d577f0aac29a5c

```

### The Flag

Finally, entering the new hash into the challenge website reveals the flag:

```bash

TCT{br34ch_th3_bl0ck_und3r_min3d_s3cur1ty}

```

![Screenshot](/static/writeups/thecyberthesisctf/crypto/blockchainbreach/3.png)

This challenge was a great reminder of the risks that come with using weak hashing algorithms like MD5, especially in systems like blockchains that are meant to be secure. It was super fun watching people struggle, figure out the patterns, and finally, crack the code!

Hope you enjoyed this walkthrough—see you in the next CTF! 🔥