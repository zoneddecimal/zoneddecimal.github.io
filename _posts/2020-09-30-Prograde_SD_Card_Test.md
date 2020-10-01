---
layout: post
title: Prograde SD Card Test
category: Tech
---

About a year ago I purchased a pair of Prograde UHS-II 250MB/s 64GB SD Cards from Amazon.  Unfortunately, I've noticed lately that when I'm using them in my X-T3 I'll occassionally encounter a card reading error, specifically when I'm taking 4k 200MB/s video.  So I figured maybe I should test them - and then it crossed my mind, are these even real Prograde SD cards?  After some searching, I find that Prograde SD cards are supposed to have a serial number on the back side of each card - mine do not!  It's possible this means that they are fake cards.

|  |  |
|--|--|
|![image](/images/20200930_220006.jpg)|![image](/images/20200930_215949.jpg)|

So right now I'm feeing a bit nervous.  Being a year out of the purchase date, it is certainly too late to return them as fakes.  I figured I would test them anyway.

## Symptoms

When using my Fujifilm X-T3 with both Prograde cards in the camera, configured in _backup_ mode for stills shooting but _sequential_ mode for video shooting, the camera will occassionally freeze during video capture with an error message about writing to the SD card.  This problem seems to happen when the video settings are configured for high quality and/or high framerate shooting.  It has definitely occurred during 200MB/s+ bitrate, 4k60p video capture.  The current firmware version is `3.21`

## Materials

I will be testing the following:
* Two identical Prograde UHS-II 300MB/s 64GB SD Cards.
* One PNY Elite Performance 95MB/s 128GB SD Card for reference.

| SD Card     | Advertised Size (GB) | Advertised Write Speed (MB/s) | Advertised Read Speed (MB/s) |
| ----------- |:--------------------:|:-----------------------------:|:----------------------------:|
| Prograde A  | 64                   | 250                           | 300                          |
| Prograde B  | 64                   | 250                           | 300                          |
| PNY Elite   | 128                  | N/A                           | 95                           |

Card Reader: 
* Kingston MobileLite G4 UHS-II USB-3 Card Reader

PC: 
* Lenovo Thinkpad T430
* Core i7-3520M
* 8GB RAM
* 128GB Toshiba SSD (Don't know the speeds, but anecdotally seem to get around 150MB/s read/write.)
* USB-3 port
* Debian (v10 Stretch) Linux

## Experiment

The three measurements I will be making are:
1. Capacity
1. Read Speed
1. Write Speed

Capacity is fairly straightforward - does the SD card actually hold the amount of data (usually measured in GB) it claims to hold?  Quite often fake SD cards will be a smaller capacity than is advertised - one way the scammers can make money off of these cards.

Read speed refers to how fast data can be taken off of the card, while write speed refers to how fast data can be put on the card, both usually measured in MB/s.  Fake cards are often quite a bit slower than their advertised speeds.  Slower cards are cheaper to make.

### Setup

In order to make the three measurements, I will be copying data from my linux laptop to the SD cards, and vice versa.

The data I will be using is randomly generated data from the /dev/urandom device (a pseudo-random number generator).  The following python script will generate 29 files, each of size 2GB, for a total of 58GB of unique data (64GB SD Cards do not come with a full 64GB of capacity).  Ensuring 58GB can fit on a card is good enough for my uses.

```python
import os
for x in range(0, 29):
    os.system('dd if=/dev/urandom of=tempfile'+str(x)+' bs=1024 count=2097152')
```

I chose 2GB file sizes since a lot of time this is the file size limit on SD cards formatted on cameras - using the FAT32 filesystem.  Run the script.

Once the data has been copied onto the card we will want to verify that the copy was successful and all data was copied successfully.  To do this, obtain the sha1 checksum of all of the files before the transfer - and then after the transfer verify that the copied files produce the exact same sha1 checksum.

```bash
>sha1sum tempfile* > sums.txt
```

The sums should look something like:

```
>head -5 sums.txt
4b43c517157cf8d20f2e86186dd7305d5d08c32c  tempfile0
93ca8718748e01fbfbbcb358339c4d061eb7b8c2  tempfile1
8532687b4585224c49c7e8c77381948d58910883  tempfile10
f54c8ebb057db1577b259912d68590d7f6840f70  tempfile11
d7c6b63947e4f08f3fff97eea6319459c9be0c2e  tempfile12
```

One last thing that is good to do for reproducible results is to format the SD cards with a fresh filesystem.  I used my camera to do that, with the `FORMAT` option.

### Copying to the SD Cards

Copying the files to the sd card is straighforward: mount the sd card to the laptop, and use

```bash
time cp tempfile* /mnt/
```

where `/mnt/` is the mount-point of the SD card.

The `time` command will produce the elapsed time that we will use to calculate the throughput.  The output will look something like:

```
real    4m36.677s
user    0m1.139s
sys     1m16.307s
```

After the copy is done, run the `sha1sum` program again and then `diff` the output with `sums.txt` to verify that the copy was successful.

Finally, copy the data back to the laptop, using the a similar copy command and record the elapsed time.  Again, run the checksum to verify the copy was successful.

Repeat this whole procedure for the other SD cards.

## Results

| SD Card    | Checksum copy to card | Checksum copy from card |
| ---------- |:---------------------:|:-----------------------:|
| Prograde A | Y | Y |
| Prograde B | Y | Y |
| PNY Elite  | Y | Y |

All of the checksums were verified, both when copying to the cards and copying from the cards.  This is great news - the advertised card sizes for the Prograde cards are indeed accurate.  (I didn't bother filling up the PNY card to verify it's capacity).

| SD Card    | Write time (s)| Write speed (MB/s) | Read time (s) | Read Speed (MB/s) |
| ---------- |:-------------:|:------------------:|:-------------:|:-----------------:|
| Prograde A | 276.7         | *225.1*            | 404.4         | *154.0*           |
| Prograde B | 297.5         | *209.3*            | 499.5         | *124.7*           |
| PNY Elite  | 1281          | *48.62*            | 685.9         | *90.80*           |

Some of the throughput speeds are somewhat surprising, although nothing jumps out at me about the Prograde cards that indicates they are fakes.  Actually, the write speeds for the Prograde cards are good and about what I would expect in actual usage.  I'm not sure why there is the big difference between the A and B cards - those results might mean another experiment is warranted.  The write speed of the PNY card seems adequate - there was no advertised write speed, only that it was "lower than the read speed".

The read speeds were definitely unexpected at the time, but I quickly realized the throughput was bottlenecked by the write speed of the internal SSD drive.  The read speed of the PNY card was close to it's advertised speed of 95MB/s.  However, the Prograde cards were quite a bit below their rated 300MB/s speeds.  Interestingly, there was still a difference between the A and B cards.

So I ran another quick test with the B card and the PNY card that read a single 2GB file into a tempfs filesystem (a filesystem in RAM, with possible paging to disk...good enough though) to test the true read speed of the cards.  I didn't do the test with the A card unfortunately, so we have somewhat incomplete results.

| SD Card    | Read time (s) | Read speed (MB/s) |
| ---------- |:-------------:|:-----------------:|
| Prograde B | 9.926         | 216.3             |
| PNY Elite  | 25.82         | 83.17             |

The read speeds into the tempfs filesystem are definitely much faster.  Curiously, the Prograde card still doesn't get close to the rated 300MB/s speed.  The PNY card performs adequately.

I think I can be fairly confident that these cards are not fakes.  The capacity is there, and the write and read speeds are sufficiently high that it is highly doubtful that scammers would use such quality SD cards.  That being said, there are still some questions that have been raised.  Perhaps these can be answered in another experiment.

1. Why is there the difference in read and write speeds between the two Prograde cards?
1. Why doesn't the Prograde B card read speed go any higher than 216.3 MB/s?
