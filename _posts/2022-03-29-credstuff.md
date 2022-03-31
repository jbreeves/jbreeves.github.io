---
layout: post
title: PicoCTF 2022 - credstuff
tags: CTF picoCTF Cryptography
---

From: picoCTF 2022 

-------

## Challenge
| Points |   Category   | 
| ------ | ------------ |
|   100  | Cryptography | 

## Description
We found a leak of a blackmarket website's login credentials. Can you find the password of the user cultiris and successfully decrypt it? 

Download the leak [here](https://artifacts.picoctf.net/c/534/leak.tar). 

The first user in usernames.txt corresponds to the first password in passwords.txt. The second user corresponds to the second password, and so on.

## Methodology
1. Download the file and extract it
    `tar -xvf leak.tar`
2. Find the line where `cultiris` is the username in `usernames.txt`. Find the corresponding line in `passwords.txt` and extract the encoded password.
    - cvpbPGS{P7e1S_54I35_71Z3}
3. Use the [monoalphabetic substitution cipher decoder](https://www.dcode.fr/monoalphabetic-substitution) in manual mode
    - We can extrapolate the first section of the flag as picoCTF. 
        - That gives us the following:
            B = O
            C = P
            G = T
            P = C
            S = F
            V = I
        - If you chart it out and start two spaces before C, you will see that the alphabet has been rotated 13 spaces (Ceasar Cipher).
       
## Flag
```
picoCTF{C##############3}
```