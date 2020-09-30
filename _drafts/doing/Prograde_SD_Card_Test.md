---
layout: post
title: Prograde SD Card Test
category: Photography
---

About a year ago I purchased a pair of Prograde UHS-II 250MB/s 64GB SD Cards from Amazon.  Unfortunately, I've noticed lately that when I'm using them in my X-T3 I'll occassionally encounter a card reading error, specifically when I'm taking 4k 200MB/s video.  So I figured maybe I should test them - and then it crossed my mind, are these even real Prograde SD cards?  After some searching, I find that Prograde SD cards are supposed to have a serial number on the back side of each card - mine do not!

<Insert card pictures>

So right now I'm feeing a bit nervous.  Being a year out of the purchase date, it is certainly too late to return them as fakes.  I figured I would test them anyway.

## Materials

I will be testing the following:
* Two identical Prograde UHS-II 250MB/s 64GB SD Cards.
* One PNY Elite Performance 95MB/s 128GB SD Card for reference.

Card Reader: 
* Kingston MobileLite G4 UHS-II USB-3 Card Reader

PC: 
* Lenovo Thinkpad T430
* Core i7-3520M
* 8GB RAM
* 128GB Toshiba SSD (Don't know the speeds, but anecdotally seem to get around 150MB/s read/write.)
* USB-3 port

## Experiment



```python
import os
for x in range(0, 30):
    os.system('dd if=/dev/urandom of=tempfile'+str(x)+' bs=1024 count=2097152')
```

