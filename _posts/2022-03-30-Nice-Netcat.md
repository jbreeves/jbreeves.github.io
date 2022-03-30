---
layout: post
title: PicoCTF - Nice Netcat
tags: CTF picoCTF Netcat
---

AcousticGirl
March 30, 2022

-------

## Challenge
There is a nice program that you can talk to by using this command in a shell: 
```
$ nc mercury.picoctf.net 43239
``` 
but it doesn't speak English...

## Netcat Session
The netcat session spits out several numbers and then disconnects.

```
└─$ nc mercury.picoctf.net 43239
112 
105 
99 
111 
67 
84 
70 
123 
103 
48 
48 
100 
95 
107 
49 
116 
116 
121 
33 
95 
110 
49 
99 
51 
95 
107 
49 
116 
116 
121 
33 
95 
55 
99 
48 
56 
50 
49 
102 
53 
125 
10 
```

## Decoding
The sequence of numbers seem to be ASCII so a small Python script can decode it.
```python
nums = [112, 105, 99, 111, 67, 84, 70, 123, 103, 48, 48, 100, 95, 107, 49, 116, 116, 121, 33, 95, 110, 49, 99, 51, 95, 107, 49, 116, 116, 121, 33, 95, 55, 99, 48, 56, 50, 49, 102, 53, 125, 10]
flag = ""
for number in nums:
    flag += chr(number)
    print(flag)
```
This spits out the flag.

## Flag
```
picoCTF{g##############################5}
```