---
layout: post
title: BLE_CTF - Write-up
---

![BLECTF]({{"/content/posts/blectf.jpg" | absolute_url }}){: .center}
<br/>

This is my write-up for the [BLE_CTF](https://github.com/hackgnar/ble_ctf) challenge by [@hackgnar](https://twitter.com/hackgnar). Overall, it took me less than a day to get everything done, including compilation, flashing, etc. and getting all the flags. The difficulty of the CTF is easy, it's simple to follow through the challenges and you can do it even if you know little/nothing about BLE. 

To do the CTF, you don't need any specialized hardware or development kit. You only need a Bluetooth adapter that supports BLE. This can be your built-in laptop chipset, an USB dongle or even your phone. I did some of the challenges from my phone with [nRF Connect](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp&hl=en_US) and it worked perfectly for simple challenges.

I did most of the challenges on my Ubuntu machine, so I invoke most of the commands with sudo. I also used aliases for a basic read/write, but I replaced these in the write-up, so you don't need to cross-reference the instructions. Making aliases is highly recommended though, unless you enjoy typing loooonnngg commands with a lot of hex, again and again and again.

<br/>

# Scanning for BLE devices

As I already [compiled the code and flashed my ESP32]({% post_url 2018-10-06-blectfflash %}), I just had to find it. *hcitool* is installed on Ubuntu by default and can be used to scan for BLE devices, so I did that first.

*bleah* is not installed by default, so I had to compile and install *bluepy* and *bleah* in order to get it working. I also encountered two bugs during runtime, so I had to fix those issues as well to get it working. If you get errors, check the github issues page first.

Both commands list the BLE devices nearby, although the *bleah* output provides more information.

    sudo hcitool lescan
    sudo bleah

<br/>

# Display score 
To get our current score, we have to read the 0x002a handle. *gatttool* can do this, but it outputs the hex values instead of the ASCII characters, so you need to do some string magic first. 

*bleah*, as far as I know, can't read from a single handle, but has an -e (--enumerate) flag that shows all the information for the handles. As this outputs a lot of information, I only did this during my initial recon.

    sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x002a | cut -d" " -f3- | sed  "s/ //g" | xxd -r -p; printf "\n"
    sudo bleah -b 24:0A:C4:08:D7:26 -e

To make my life easier, I set up a temporary alias for the *gatttool* command:
    
    alias getscore="sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x002a | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'"

<br/>

# Send flag 
Main difference between the two is that *gatttool* requires you to hexdump the ASCII, while *bleah* does this for you automatically. 
    
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-write-req -a 0x002c -n $(echo -n "12345678901234567890"|xxd -ps)
   sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "12345678901234567890"

For this usecase I prefered *bleah*, so I set up another alias:

    alias submitflag="sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d $1"

<br/>
# Challenges - spoilers ahead!

As I checked the score, I knew that there are 20 flags total. The *bleah -e* command also revealed the text from all of the handles, so I decided to go through the challenges one-by-one.  


#### Flag 1 - This flag is a gift and can only be obtained from reading the hint!
The first flag is the odd-one-out, as you can only get this flag if you reveal the hint in the Github repo. Opening the hint revealed the flag/command that I needed to submit. 

    sudo gatttool -b 24:0a:c4:08:d7:26 --char-write-req -a 0x002c -n $(echo -n "12345678901234567890"|xxd -ps)

<br/>

#### Flag 0x002e - Learn how to read handles
The second flag was even easier. As I did *bleah -e* earlier, I already saw the value of this text field, so I submitted it and scored a point. 

    sudo gatttool -b 24:0a:c4:08:d7:26 --handle=0x02e --char-read | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p | cut -c-20; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "d205303e099ceff44835"

<br/>

#### Flag 0x0030 - Read handle puzzle fun
This one is also easy. When I scanned for the device, it already revealed its name, which was BLECTF. I simply ran md5sum on the name, truncated it to 20 characters and submitted the flag. When you echo remember to add the -n option, otherwise it adds the newline char to the string **before** hashing it.

    echo -n "BLECTF" | md5sum | cut -c-20
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "5cd56d74049ae40f442e"

<br/>

#### Flag 0x0016 - Learn about discoverable device attributes
This flag also revealed itself with *bleah -e*, so I simply fetched the value, truncated it 20 characters and submitted it. Score!

    sudo gatttool -b 24:0a:c4:08:d7:26 --handle=0x016 --char-read | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p | cut -c-20; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "2b00042f7481c7b056c4"

<br/>

#### Flag 0x0032 - Learn about reading and writing to handles
The text revealed that I had to write anything to the 0x0032 handle, so I used *bleah* to submit the string anything. After submitting the value, the text changed, revealing the flag.

    sudo bleah -b 24:0a:c4:08:d7:26 -n 0x0032 -d "anything"
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x0032 | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "3873c0270763568cf7aa"

<br/>

#### Flag 0x0034 - Learn about reading and writing ascii to handles
This one is more or less the same as the previous, only now, you need to submit 'yo'. After I submitted the message, I queried back the value and the flag revealed itself. Nice!

    sudo bleah -b 24:0a:c4:08:d7:26 -n 0x0034 -d "yo"
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x0034 | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "c55c6314b3db0a6128af"

<br/>

#### Flag 0x0036 - Learn about writing hex to handles
Same as before, but now I used *gatttool* to submit the value, as it uses hex by default. 

    sudo gatttool -b 24:0A:C4:08:D7:26 --char-write-req -a 0x0036 -n 07
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x0036 | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "1179080b29f8da16ad66"

<br/>

#### Flag 0x0038 - Learn about writing to handles differently
Again, no magic here. The handle tells us to write 0xc9 to handle 0x0058 and once I did that, the handle changed it's value.
  
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-write-req -a 58 -n c9
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x0038 | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "f8b136d937fad6a2be9f"

<br/>

#### Flag 0x003c - Learn about write fuzzing
Fun time starts now. The handle asks us to brute force it's value from 0x00 to 0xFF. As I am too lazy to manually send 256 commands, it's time to call in Python. 

Nothing crazy in the script, we start a for loop where i counts from 0 to 255. As 0 is 0x00 and 255 is 0xff, if we use the format function to convert our int to hex string and zfill to pad it to two digits, we can easily transform i into our values. Now all I needed to do was to concatenate the command and the hex value and pass it to the os.system function.  

fuzz.py:

    #!/usr/bin/python
    import os
    for i in range(256):
        hex = format(i,"x").zfill(2)
        program = 'sudo gatttool -b 24:0A:C4:08:D7:26 -a 0x003c --char-write-req -n ' + hex
        os.system(program)

To complete the challenge I executed the script, fetched the value of the handle and submitted it as the flag.

Commands:

    python ./fuzz.py
    sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x003c | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "933c1fcfa8ed52d2ec05"

<br/>

#### Flag 0x003e - Learn about read and write speed
This challenge is easier than the write one. We simply have to read the value a 1000 times, so I wrote a small script that does that and I submitted the flag manually.

read.py:

    #!/usr/bin/python
    import os
    for i in range(1001):
        os.system("sudo gatttool -b 24:0A:C4:08:D7:26 --char-read -a 0x003e | cut -d' ' -f3- | sed 's/ //g' | xxd -r -p; printf '\n'")

Commands:

    python ./read.py
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "6ffcd214ffebdc0d069e"

<br/>

#### Flag 0x0040 - Learn about single response notifications
The handle revealed that I need to sign up for response notifications to get the flag. By writing 0x0100 to the handle and setting the listen switch in *gatttool* I received the flag in no time.

    sudo gatttool -b 24:0a:c4:08:d7:26 --char-write-req --handle=0x0040 --value=0x0100 --listen
    echo "35 65 63 33 37 37 32 62 63 64 30 30 63 66 30 36 64 38 65 62" | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "5ec3772bcd00cf06d8eb"

<br/>

#### Flag 0x0042 - Learn about single response indicate
Same as before, but now we need to set 0x0200 as the value for the write.

    sudo gatttool -b 24:0a:c4:08:d7:26 --char-write-req --handle=0x0044 --value=0x0200 --listen
    echo "63 37 62 38 36 64 64 31 32 31 38 34 38 63 37 37 63 31 31 33" | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "c7b86dd121848c77c113"

<br/>

#### Flag 0x0046 - Learn about multi response notifications
Same as the 0x0040 flag. Setting up listen in *gatttool* doesn't terminate the connection after the first response, so I left it running and received the flag after a few seconds. 

    sudo gatttool -b 24:0a:c4:08:d7:26 --char-write-req --handle=0x0046 --value=0x0100 --listen
    echo "63 39 34 35 37 64 65 35 66 64 38 63 61 66 65 33 34 39 66 64" | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "c9457de5fd8cafe349fd"

<br/>

#### Flag 0x0048 - Learn about multi response indicate
Same as the previous challenge, but now with the value 0x0200.

    sudo gatttool -b 24:0a:c4:08:d7:26 --char-write-req --handle=0x004a --value=0x0200 --listen
    echo "62 36 66 33 61 34 37 66 32 30 37 64 33 38 65 31 36 66 66 61" | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "b6f3a47f207d38e16ffa"

<br/>

#### Flag 0x004c - Learn about BT client device attributes
To get this flag, you have to connect with the MAC address of 11:22:33:44:55:66. bdaddr can change the MAC if your device supports it, but it didn't work with my built-in Bluetooth device. As I couldn't find my dongle, I just submitted the flag from the source. ¯\\_(ツ)_/¯

    sudo ./bdaddr -i hci0 -r 11:22:33:44:55:66
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "aca16920583e42bdcf5f"

<br/>

#### Flag 0x004e - Learn about message sizes MTU
To get this flag, I had to change my MTU to 444. The *gatttool* mtu switch didn't trigger the flag, so I used the interactive mode and read the flag from handle 0x004e.

    sudo gatttool -b 24:0a:c4:08:d7:26 -I
    > connect
    > mtu 444
    > char-read-hnd 0x004e

    echo "62 31 65 34 30 39 65 35 61 34 65 61 66 39 66 65 35 31 35 38 " | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "b1e409e5a4eaf9fe5158"

<br/>

#### Flag 0x0050 - Learn about write responses
In theory you should get a response back after writing hello to the handle with char-write-req. Somehow I didn't get the write response, however reading the handle again granted me the flag.

    sudo gatttool -b 24:0a:c4:08:d7:26 -a 0x0050 --char-write-req --value=$(echo -n 'hello' | xxd -p) 
    sudo gatttool -b 24:0a:c4:08:d7:26 -a 0x0050 --char-read
    echo "64 34 31 64 38 63 64 39 38 66 30 30 62 32 30 34 65 39 38 30 00 " | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "d41d8cd98f00b204e980"

<br/>

#### Flag 0x0052 - Hidden notify property
As both the text and *bleah -e* revealed, this handle has no notify property. No problemo, I connected with 0x0100 and --listen and received the flag.
    sudo gatttool -b 24:0a:c4:08:d7:26 -a 0x0052 --char-write-req --value=0x0100 --listen
    echo "66 63 39 32 30 63 36 38 62 36 30 30 36 31 36 39 34 37 37 62" | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "fc920c68b6006169477b"

<br/>

#### Flag 0x0054 - Use multiple handle properties
This handle has multiple properties, so it makes sense to poke around to see if it returns the flag. I got the first flag by writing to the handle and the second, by setting up notifications and listening. As both flag fragments were 10 byte long, I simply concatenated them.

Flag part 1:

    sudo gatttool -b 24:0a:c4:08:d7:26 -a 0x0054 --char-write-req --value=4141
    sudo gatttool -b 24:0a:c4:08:d7:26 -a 0x0054 --char-read
    echo "66 62 62 39 36 36 39 35 38 66" | sed "s/ //g" | xxd -p -r; printf '\n'

Flag part 2:

    sudo gatttool -b 24:0a:c4:08:d7:26 -a 0x0054 --char-write-req --value=0x0100 --listen
    echo "30 37 65 34 61 30 63 63 34 38" | sed "s/ //g" | xxd -p -r; printf '\n'
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "fbb966958f07e4a0cc48"

<br/>

#### Flag 0x0056 - OSINT the author!
Nothing fancy, find the Twitter handle, md5sum and truncate it to 20 chars, then submit the flag. 

    echo -n "@hackgnar" | md5sum | cut -c-20
    sudo bleah -b 24:0A:C4:08:D7:26 -n 0x002c -d "d953bfb9846acc2e15ee"

<br/>

# The end

I really enjoyed the CTF and I learned an awful lot about BLE. The challenges teach a lot about hands-on interaction with BLE devices and I feel like I have a great starting point to continue this adventure. My next step will be to hit up a reference or a book about development, so I can learn the foundations. Remember kids, you have to understand stuff, in order to break stuff. 
