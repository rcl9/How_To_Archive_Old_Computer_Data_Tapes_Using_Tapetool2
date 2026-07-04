# How To Archive Old Computer Data Tapes Using Tapetool2 (for the Exidy Sorcerer)

This repository captures  my experience with retrieving and archiving early 1980's Exidy Sorcerer data tapes and directly converting them to binary, Exidy Sorcerer .tape format and a related de-blocked text files. If I had access to this repo before starting my project then it woud have saved me a good day or two of head banging and head scratching. 

<div style="text-align:center">
<img src="/Images/Tutorial.webp" alt="" style="width:75%; height:auto;">
</div>

## A Quick Overview of TapeTool2

As explained on their [WEB site](https://www.toptensoftware.com/tapetool), TapeTool2 is a command line utility for converting and repairing Microbee, TRS-80 and Sorcerer audio tape recordings with the main objective of recovering lost data. TapeTool is not an automatic data recovery/repair tool - rather it should be considered a toolkit of useful utilities for data extraction, repair and re-rendering of working audio tape recordings.

The main problem with this (excellent) utility program is that it does not provide any new-user information about how it really needs to be executed. In retrospect, it's a pretty easy process to follow but not entirely obvious from a first read. It is a "pipeline" tool that depends heavily on how that audio and data processing pipeline is set up and that is not well documented.

## My Objectives

From the late 70's up until late 1981 I had to use the 1200 baud cassette data interface on the Exidy Sorcerer to store all of my software, including the two gruelling years I had spent writing "[Sorcerer Space Invaders](https://github.com/rcl9/Resurrecting-and-Rebuilding-Sorcerer-Space-Invaders-from-1980--BASEX-)" in BASIC and then [BASEX](https://github.com/rcl9/History-Of-The-BASEX-Compiler-from-the-1979-Era). I had converted all of my tape-based software collection from tape to 8" disk-based CP/M by early 1982 but not my Space Invaders development environment + files. There may have been reasons that I had chosen not to go through the tedious task of archiving a hundred tape-based files at that time, knowing what I do today. 

My objective was to "try" and retrieve all of the BASEX and related source files from four 50 year old audio tapes without using my original Exidy Sorcerer to do so. I didn't think it would be possible but I ideally wanted an entirely digital pipeline to take the raw audio tapes into coresponding binary and/or .tape files which I could then mount in the MAME or jSorcerer emulators, and/or to rebuild my original Sorcerer Space Invaders development pipeline. Amazingly, and utterly unbelievably, I did achieve these goals but not without a lot of head banging and sheer determination. 

As a side note, this process also taught me just how much you can forget years or decades later, and also what memories + prior processes can be resurrected through careful digital archaeology. I started with almost nothing in hand yet was able to fully rebuild my Sorcerer Space Invaders development pipeline. Knowing about this problem of "fading memories", over the last 15 to 20 years, I now actively document everything I do and "where I put things" digitially, as I've learned how quickly the human memory throws out current context (ie. 3 years is the magic number). 

## Step 1 - Audio Data Capture

I started with 45 old Exidy Sorcerer (low quality) audio data cassette tapes in hand (1200 baud) and no other knowledge of what may be on them nor of how to fully restore the same development environment I was using 45 years ago.

The first step was to capture .wav audio files from the tapes. This took a few days to do properly. Forunately I still had my original Sears cassette recorder which I had used to create the tapes back in the day. I cleaned the heads and capstan using isopropyl alcohol. I also fully  wound the tapes from start to end and back again just to "loosen them up a bit".

The *Mic* output of the recorder was connected to the *Line-In* of the PC. From prior projects I jumped at using WavePad by NCH Software which I've found to be a pleasure to use, and does not require reading any manual. I chose to record around -8db based on experience. After each audio segment was recorded I trimmed the start and ending before saving out to an uncompressed PCM .wav file in MONO format. Do not forget to choose MONO!

Overall I found that the data on two tapes sounded (to me) fairly consistent and okay. However, one original tape (which may not have been made by me) was choppy while another one tended to have issues for the first few files. I'd quickly find out that this was true when trying to turn the audio back into error-free digital data. 

## Step 2 - Audio to Digital Transformation

This phase was a lot more tricky and with no guaranteed results in advance. Fortunately I was able to retrieve a high percentage of the files error free (based on their CRCs and also some binary file comparisons).

My end result was the script named [convert.btm](</Batch files/convert.btm>) which uses the "4DOS/4NT/TCE/Take-Command" scripting language by [JP Software](https://jpsoft.com). This encapsulates my process by which I automated the testing, tweaking and conversion of each .wav file to a binary (.bin), Exidy Sorcerer .tape file and a corresponding text file of what Sorcerer "tape blocks" were found on the tape. I ended up running this script many hundreds of times until I could get a set of error free conversions. 

The first processing stage is to execute TapeTool2 to do these operations:

1. Take the raw audio file (%1) and apply bandpass filtering + noise smoothing.

1. Parse the audio as a 300/1200 baud stream into a raw byte stream

1. "De-block" the raw byte stream using the Exidy tape format CRC-based block-based format. This becomes the "block stream" for TapeTool2

1. Write a text file (%~n1.txt ) of the blocks and header info found in the file, especially the loading address + execution address which is super critical for post-data analysis. 

1. Save out a binary image (%~n1.bin) of the file which can be executed later on via CP/M. Note: you would need to offset it to the starting address relative to 0x100.

```
tapetool2 --sorcerer %1 ^
	bandPass parseAudio --baud:1200 --noiseThreshold:%NOISE_THRESHOLD% ^
	textBlockStreamWriter --filename:%~n1.txt ^
	unpackData %~n1.bin
```

I chose to segment out a second pass on the audio data to create the Exidy Sorcerer .tape file which is basically a 100% digital byte copy of the cassette data stream (in Exidy "block" format + CRCs):

```
tapetool2 --sorcerer %1 ^
	bandPass audioToBytes --baud:1200 --noiseThreshold:%NOISE_THRESHOLD% ^
	out.bin
```

In both of these examples you will notice that I am using a bandpass filter and noise filtering. Through my experience I found that the bandpass filter was absolutely necessary but that the noise filtering most often lead to failed de-blocking. Hence, using IF/ELSE statements in the script file, I set up another code path which would drop the use of noise filtering via these two defines:

```
	set USE_NOISE_THRESHOLD= 0
	set NOISE_THRESHOLD=0.1
```

I had originally done my data processing using a noise threshold of 0.05, 0.07 or 0.1 but came to realize that the failed conversions were mainly due to the use of the noise filtering. I generally had more immediate and error-free success by disabling the noise filtering.

In general, if my ear could hear "audio inconsistencies" or my eyes saw volume fluctuations/dropouts then those tapes could not be de-blocked properly. I did try audio normalization and other methods but alas, in my simplistic approach, I was not able to retrieve those audio recordings. 

In this example we can see a sudden drop in audio level:

<div style="text-align:center">
<img src="/Audio waveform snapshots/glitch1.webp" alt="Glitch #1" style="width:70%; height:auto;">
</div>

And the following is an ideal "worst case" scenario that I had encountered whereby the audio levels were wildly varying and there was frequency shifts in the audio tones. Ideally this audio waveform should be consistent all the way through. I suspect that this tape was written by someone else as it was clearly not of the same (overall quality) as my other data tapes:

<div style="text-align:center">
<img src="/Audio waveform snapshots/glitch2.webp" alt="Glitch #2" style="width:70%; height:auto;">
</div>

And likewise with this example. 

<div style="text-align:center">
<img src="/Audio waveform snapshots/glitch3.webp" alt="Glitch #3" style="width:70%; height:auto;">
</div>

After running my script file on an audio file I had to visually watch out for one of two runtime errors whereby the de-blocking process failed. If it passed then I knew the CRC-based error checking was successful and that the resulting data was good. 

A key point of interest, when using TapeTool2, is that a "byte stream" is the basic, reconstructed byte-aligned data stream from the audio data. You can then execute various de-blocking algorithms (ie. for the Exidy Sorcerer) which takes the byte stream and turns them into "block streams". Figuring out which pipeline filter to use was a trial and error process given that there is no documentation other than using the "-h" command line option to display the syntax for each filter module. However, after a while it starts to all make sense. 

## Loading the .tape Files in to an Exidy Sorcerer Emulator

I had been using MAME to emulate CP/M on the Sorcerer (using the Dreamdisk controller emulation) but I had no luck loading up my newly archived .tape files. Back in 2011 I had helped the author of the then-new jSorcerer emulator with dozens of my archived Exidy Sorcerer game files and hence decided to give it a spin again. I was super-surprised to see how much easier it was to use (entirely mindless) compared to using MAME. 

Once the .tape files have been created, they can be quickly and easily loaded up into jSorcerer in a jiffy using this method:

1. Execute the emulator via "sorcerer.jar" 

1. Click on the first Cassette icon on the right side. Choose a .tape file. 

1. Type "LO" into the Exidy monitor in upper case.

	- After a few second you should see `Found -- <<name>>`

	- If there are any issues then either use "SE T=0" or "SE T=1" to set the 300/1200 baud tape speed.

1. Take note of the reported "GOADDR" upon final loading and then do `GO <address>` to execute

Note: the Exidy cassette tape block file format can be found at page 17 of the [Exidy Software Internals Manual 1979 by Tolomei Vic](https://archive.org/details/Exidy_Software_Internals_Manual_1979_Tolomei_Vic/page/n17/mode/2up).

## Step 3 - Using the Files For Data Archaeology

Retrieving the audio data and turning the resulting files into 100% error free (CRC verified) digitial data files was only step one. It was similar to that of uncovering a new archeology find from ancient Egypt which provided few answers or clues as where and what it represented. Likewise, having my BASEX and Sorcerer Space Invaders digital files in hand was a major first step but provided no clues as to how BASEX worked back in the day or how the 4 key files were related to each other for code execution. That would be another major [adventure by itself](https://github.com/rcl9/History-Of-The-BASEX-Compiler-from-the-1979-Era). 

## See Also

[TapeTool2](https://www.toptensoftware.com/tapetool) by Topten Software

[TapeTool - A Microbee Tape Diagnotic and Recovery Utility (older open source version)](https://github.com/toptensoftware/tapetool)

[Exidy Software Internals Manual 1979 by Tolomei Vic](https://archive.org/details/Exidy_Software_Internals_Manual_1979_Tolomei_Vic/page/n17/mode/2up)

[Cassette: Load Exidy Sorcerer software from a WAV file (Python based)](https://github.com/robjordan/cassette)

[Storing and Loading Exidy Sorcerer Programs From Cassette Tapes](https://web.archive.org/web/20260211031055/https://www.trailingedge.com/exidy/exbasic11.html)
