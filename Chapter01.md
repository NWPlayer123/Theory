# Chapter 01: "Rules" of the Trade
Hello, I assume you're here cause you wanna pick up some tips or info. Well, this would be the place, outside of straight up asking me on [Twitter](https://twitter.com/NWPlayer123).

## Rule Number 1: Know Your Environment
This seems straightforward but it helps you make a lot of assumptions to narrow down how the thing is actually working
- General
  - Assumption 1: if it works normally when you play/use it, you can replicate that behavior.....eventually, you just need to figure out what it's doing to get that behavior.
- Nintendo 3DS
  - Overview: Data Structure
    - Storage format is usually CIA/3DS (cartridge dumps but we convert to CIA to install/decrypt)
      - Usually just 00000000 and 00000001, 0 has the game (ExeFS and RomFS), 1 has the Manual (mandatory on the 3DS)
      - Content numbers can vary but it's always(?) two contents total
      - Base game (00040000) is almost always 00000000 and 00000001, updates are what change up numbering based on how many are pushed, they also might only have one, only need to update the game, not the manual
      - One exception is DLC which has its own unique structure
        - 00000000 has the metadata (MetaDataContentHeader.bin, icon.bin, ContentInfoArchive_USA_en.bin, icons/*.icn)
        - All packages with actual data are 00000001+, and as you purchase stuff from the eShop, it only downloads those specific packages so you don't get all DLC from one purchase, but of course they all use the same key on CDN so we can buy one and be able to get them all for poking at
  - Assumption 1: It's a mostly-closed system, games can't really download data over the internet and store it. Outside of a bit of save data and such, developers have to either leave it as-is or push a full-on update to change stuff, so once you grab the base game and the latest update, you can assume you have all needed data.
  - Assumption 2: no ASLR/code address changing, what you see in the output from [ctr-elf](https://github.com/archshift/ctr-elf) or [ctr-elf2](https://github.com/NWPlayer123/ctr-elf2) is correct in the actual 3DS memory
  - Assumption 3: As far as I've seen, .code/.text always starts the same way, BL sub_100024, which is a function to set up the .bss section, hardcoded size and it just does a memset(0) to make sure there's no "bad"/unexpected data, and after that does more setup for the program to actually run
    - The last instruction/branch in the list should always be SVC 3 which is [ExitProcess](https://www.3dbrew.org/wiki/SVC#System_calls) in case something terrible happens when it runs
    - Therefore, the one behind it should(?) be main()
- Android/Mobile
  - Assumption 1: See General Assumption 1, Android is a lot more flexible than 3DS, apps can download and store whatever the hell they need, but, assuming it works when ran normally, you can replicate that behavior, even if it takes a lot of shit to get working.
  - Usually, though, all you want is the assets which they'll either cache as you play, or do a bulk download on first boot (like Fire Emblem Heroes, which actually has two, one for tutorial and one for the full game), or be self-contained in the APK (like Magikarp Jump, as far as I've seen).
    - Good way to get this working, assuming the app doesn't specifically check and break in it, [BlueStacks](http://www.bluestacks.com) lets you run it to do the bulk download/caching. It's surprisingly compatible so if something breaks and it's a big enough app, it's probably cause it's checking for the emulator and breaking.
  - Expectations for file formats
    - Most games I've run into use Unity for their app, I use [Assets Bundle Extractor](https://7daystodie.com/forums/showthread.php?22675-Unity-Assets-Bundle-Extractor) and open "globalgamemanagers" (no extension) which links all files, as far as I've seen. Some games like to break so you might have to go thru manually and extract.
    - Aside from looking if that file exists somewhere in /assets, the folder also has a bunch of hex-named files (e.g. 00b10794f8cfe0543aa50f54bde4c93e) which hold individual(?) files, ABE can also open these individual files (and multiple at once, Shift+Click in list to highlight multiple) and extract them, which should work when opening globalgamemanagers fails to extract some stuff
    - What you're looking for in ABE is Texture2D which is all images, model textures and UI and etc, AudioClip, which can hold both SFX and fully-streamed music, and TextAsset which may or may not be encrypted, but obviously holds text data. I haven't tried extracting models but those should export fine too. You'll want to use its plugins when you Shift+Click to highlight all of one type (e.g. Texture2D), cause it'll let you export to TGA/PNG, TXT, and WAV.
    - Some games take their own approach, like Magikarp Jump, which just does a xorpad on files to scramble them, you can find my script [here](https://gist.github.com/NWPlayer123/23704bc2ba9c863fd3e11068c49948c5) (half-assed, should convert almost all files, I forgot to add .ttf support), and then the filenames are embedded in the .so library, which it MD5 hashes to get the actual file.
    - Fire Emblem Heroes, as mentioned above, on first boot, downloads like 90MB for the tutorial, you need to power thru that, and then it'll download the full game, uses WEBP format to store its images, you can convert with something like [XnConvert](http://www.xnview.com/en/xnconvert/) (which I recommend for basically anything image-related)
    - To access root in BlueStacks to get its cached/downloaded files, do this:
      - Grab adb from [here](https://developer.android.com/studio/releases/platform-tools.html) and put it somewhere safe, I put it in C:\Android\SDK\platform-tools since that's where it installs normally
      - !!! BlueStacks has to be open and running (not an app, just GUI/engine working) !!!
      - ```
        adb shell
        /system/xbin/bstk/su (obtain SuperUser/root to access the files to copy them)
        pm list packages (if you don't know package name, it's in the App Store url, e.g. [jp.pokemon.koiking](https://play.google.com/store/apps/details?id=jp.pokemon.koiking&hl=en)
        cd /data/app/
        cp *packagename*-1.apk /sdcard/windows/BstSharedFolder (if you want to grab a clean APK)
        cd /data/data/*packagename* (has the files you want/it downloaded, usually wanna then "cd files")
        cp -R -f * /sdcard/windows/BstSharedFolder (this does recursive, forced, copy to the BlueStacks Shared Folder, which as of writing, is at "C:\ProgramData\BlueStacks\Engine\UserData\SharedFolder" (ProgramData is hidden in Windows by default, copy-pasting path should work))
