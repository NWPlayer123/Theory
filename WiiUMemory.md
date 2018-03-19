Someone's been asking me about how the Wii U works so I figured I should do a write-up for anyone else that stumbles on it.

# Good Links

https://fail0verflow.com/blog/2014/console-hacking-2013-omake/

https://github.com/NWPlayer123/CafeOS/blob/master/wiiu.cfg

http://wiiubrew.org/wiki/Cafe_OS#Virtual_Memory_Map

# Write-Up Part ?

To start off with, there's two CPUs, the Starbuck, which is an ARM926EJ-S (ARMv5TEJ instruction set if you're loading it into IDA or whatever), and the Espresso, which is a 3-core PowerPC CPU that idk what it's based on, with Paired Singles like the GameCube and Wii, and a bunch of custom SPRs (see link above).

The Starbuck is basically all security, with the Espresso doing all the magic. I guess the main thing to understand is how things are set-up. On boot,
* The on-dye bootrom boot0 is loaded
* which does init, loads boot1, and locks its decryption key
* boot1 does the rest of init, including load fw.img into memory
* fw.img in OSv10 is "IOSU" which does its own init, my knowledge is fuzzy here
* AFAIK IOSU loads in the PowerPC kernel, and it does its own init.
* Eventually you get everything loaded in and the home menu is ran.

When you're launching an application, there's a lot of stuff that happens.
* loader.elf in OSv10 is ran, which loads the application into memory,
* decompresses the RPX, working with the PPC and ARM kernels
* loads in any necessary RPLs(?)
* sets up permissions and such from the app, cos, meta.xml
* passes control over to the application

Wii U memory has two different modes, Physical Mode and Virtual Mode. AFAIK, IOSU does all its work in physical mode, and it can see all memory. The Espresso has to ask the Starbuck if it wants to use physical memory, OSEffectiveToPhysical syscall, both the kernel and any applications. Espresso works in Virtual Mode, which you can see from the third link.

* 0x01000000-0x01800000 (0x800000) is where all dynamic RPLs are loaded, both OS ones and any the application has(?)
* 0x0???????-0x10000000 (depends per application) is where .text of the RPX is loaded, this is the only executable memory
* 0x10000000-0x50000000 (0x40000000) is dedicated to application data, all other RPX sections are loaded in here, as Read/Write(?)
* 0xF0000000-0xFFFFFFFF (0x0FFFFFFF) is where the kernel lives, and is protected from read/write from "userspace" applications, have to use syscalls

Wii U applications also get some additional memory, I don't remember it all off the top of my head but they get a section of super-fast memory for housing graphics or whatever they want, and some other memory, but 0x10-0x50 is where all the good stuff goes on.

