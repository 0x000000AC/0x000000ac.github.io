---
layout: post
title: Nano Syntax Highlighting for C with Mac
---

Recently, I have rekindled an interest with the C programmming langauge and have decided that I would
pick up the canonical [K&R book](http://www.amazon.com/C-Programming-Language-2nd/dp/0131103628/ref=sr_1_1?ie=UTF8&qid=1419995398&sr=8-1&keywords=C+programming) on the subject.  
Most that I know who didn't take any formal classes on the subject recommend it.

I'm fond of editing on the go with my laptop -- which for now is a Macbook pro.   From experience around Linux sysadmins
I knew that may edited their bashrc files for their user directory so that they could have special text highlighting
and a plethora of other shortcuts.  Seeing that, and spending some time in Visual Studio made me want to have some
syntax highlighting for C in bash.   Instead of eding the bashrc or profile though, I went with editing the nanorc.

Here's how I did it:

1. sudo mkdir /usr/local/share/nano
2. sudo nano /usr/local/share/nano/c.nanorc
3. I pasted in a template for C highlighting that I found on Google Code. [Here](https://code.google.com/p/nanosyntax/source/browse/trunk/syntax-nanorc/c.nanorc)
4. nano ~/.nanorc
5. I inculded a reference to the file I created: include "/usr/local/share/nano/c.nanorc"


![Example](/images/Nano.jpg)
