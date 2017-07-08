---
layout: post
tags : daily
premalink: pretty
title: These days
---

突如其来的就想用英语写点东西了，好，就用英语写篇不算日记的日记吧。

Let's talk about what i have done these days...

First, I took part in a CTF game and tried to figured out a problem which ask me to find a flag in a broken disk image.And, my disk was broken....

Second, my GPT(GUID Partition Table) was broken and I lost lots of files.

Third, I changed my HHD to a SSD.

Not all yet.

Finally I found all my files without filenames....OMG!

Can you imagine that thousands of files laying there and the filenames are totally numbers? Yes, I can.

Filename maybe very important to windows system but for me, a ubuntu user, it's not a matter. Because the system can not only identify the file by its name but also its file head. That's to say, the filename is not necessary to it, usually it's for our human beings to recognize it. And luckily I got a *file* command to recognize the file. So, I can pick up all the JPG files and all the PNG files and the others.

But, they still didn't have filename extensions.

Here is my script to tell them apart. Quiet stupid... And I have to "ls" the filenames to a text file and split them in four or more to run more scripts at the same time.

	#!/bin/bash
	while read -r line
	do
	    file_name=$line

	file_info=`file -b $file_name`

	if [[ `echo $file_info | grep 'JPEG image data'` ]]; then
	    mv $file_name $file_name.jpg
		elif [[ `echo $file_info | grep 'gzip'` ]]; then
		    mv $file_name $file_name.tar.gz
		elif [[ `echo $file_info | grep 'Matroska data'` ]]; then # mv 2 floders
			mv $file_name Matroska_data/
		elif [[ `echo $file_info | grep 'ISO Media, MPEG v4 system, iTunes AAC-LC'` ]]; then
				mv $file_name ISO_media_iTunes/
		elif [[ `echo $file_info | grep 'kbps'` ]]; then
				mv $file_name music_kbps/
		elif [[ `echo $file_info | grep 'PNG image data'` ]]; then
			mv $file_name $file_name.png
		elif [[ `echo $file_info | grep 'HTML document'` ]]; then
			mv $file_name $file_name.html
		elif [[ `echo $file_info | grep 'PHP script'` ]]; then
			mv $file_name $file_name.php
		elif [[ `echo $file_info | grep 'C++ source, ASCII text'` ]]; then
			mv $file_name $file_name.cpp
		elif [[ `echo $file_info | grep 'C source'` ]]; then
			mv $file_name $file_name.c
		elif [[ `echo $file_info | grep 'GIF image data'` ]]; then
			mv $file_name $file_name.gif
		elif [[ `echo $file_info | grep 'XML document text'` ]]; then
			mv $file_name $file_name.xml
		elif [[ `echo $file_info | grep 'Microsoft Word'` ]]; then
			mv $file_name $file_name.doc
		elif [[ `echo $file_info | grep 'Microsoft Excel'` ]]; then
			mv $file_name $file_name.xls
		elif [[ `echo $file_info | grep 'directory'` ]]; then
			mv $file_name directorys/
		elif [[ `echo $file_info | grep 'PDF document'` ]]; then
			mv $file_name $file_name.pdf 
		elif [[ `echo $file_info | grep 'Zip archive data'` ]]; then
			mv $file_name $file_name.zip
		elif [[ `echo $file_info | grep 'Composite Document File V2 Document'` ]]; then
			mv $file_name $file_name.doc
		elif [[ `echo $file_info | grep 'UTF-8 Unicode text'` ]]; then
			mv $file_name $file_name.txt
		elif [[ `echo $file_info | grep 'Audio file with ID3'` ]]; then
			mv $file_name $file_name.mp3
		elif [[ `echo $file_info | grep 'ISO Media, MPEG v4 system, 3GPP'` ]]; then
			mv $file_name $file_name.3gp
		elif [[ `echo $file_info | grep 'ISO Media, MPEG v4 system, version'` ]]; then
			mv $file_name $file_name.mp4
		elif [[ `echo $file_info | grep 'GIMP XCF image data'` ]]; then
			mv $file_name $file_name.gimp 
		elif [[ `echo $file_info | grep 'PE32 executable (console)'` ]]; then
			mv $file_name $file_name.exe 
		elif [[ `echo $file_info | grep 'PE32 executable (DLL)'` ]]; then
			mv $file_name $file_name.exe 
		elif [[ `echo $file_info | grep 'Microsoft Cabinet archive data'` ]]; then
			mv $file_name windows_archive/
		elif [[ `echo $file_info | grep 'RAR archive data'` ]]; then
			mv $file_name $file_name.rar
		elif [[ $file_info == "data" ]]; then
			mv $file_name data/
		elif [[ $file_info == "ASCII text" ]]; then
			mv $file_name ascii_test
		elif [[ $file_info == "SO-8859 text" ]]; then
			mv $file_name so-8558-text
	else
		echo $file_info >> file_info
	fi
	done < $1

It's really a hard job...really...

Till now...

Now let's saying something about the SSD. One word -- fast! Only 30 seconds to startup (Notice that I encrypted my disk). And only one seconds to start Firefox and less for Chrome~

There are many tutorials to teach you what to do after setup an SSD, so, here is nothing to teach.

One more, you must format the SSD first or you will waste lot of time to realize that. Remember, the factory won't do that for you.

I also buy a disk enclosure to put my HDD in it. Then, I got a portable hard drive, I can put the backups there.

These days I also tried to install more OS to the portable hard drive, but except Ubuntu, all my tries failed. Maybe it's a wrong decision, so, I quit. Using Kali via  Virtual box  is not that worse, after all I got a SSD.  :)

And tonight I found an interesting thing that I can write a space in markdown~ Oh, not that space, it's, em... let me tell you the difference:

* That is a space: " "; 

* That is another kind of space: "　"; 

So, can you see that?

Of cause not. Because I edit them in hex editor. Which the first space is "20" and the second one is "E3 80 80". If you search "E3 80 80 ", you'll get more.[Here](http://www.utf8-chartable.de/unicode-utf8-table.pl?start=12288&number=128) is the link.

And you can copy the two "spaces" to a text editor and reopen it by a hex editor after you save it. You can see that one is "20", and the other is "E3 80 80".

Markdown only regard "20" as a space but in Unicode, "E3 80 80" is also a space while markdown didn't know that. So, we can do many things via this "characteristic", for example, XSS. Actually, many other languages also didn't know "E3 80 80" or the other Unicode brothers. And just now [he](http://joshuaghost.github.io/) told me that there was a serious bug in Unicode. Sad..

Okay, 1:31 am now.

Time to sleep.
