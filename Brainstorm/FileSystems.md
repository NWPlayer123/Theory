## Interfacing
Thinking about C++ before jumping in again, one of the important functions is reading and writing files, memory can only hold so much

### Standard
When compiling for normal systems (Windows, Mac, Linux, probably others), it seeeeeeems like you just use std::fopen, which the compiler converts to actually interface with the OS, so you don't have to fuxx with syscalls and make it super non-portable

## Embedded
Since embedded systems, by their nature, don't work the same as the major OSes, so whoever made the OS needs to provide their own compiler and interface to let you read files, which only needs a few certain functions, rest the program can make on its own:
- Directories
  - Move, Create, Delete, Rename, Copy (optional?)
- Files
  - Move, Create, Delete, Rename, Copy
  - Open, Close
    - Seeking in file
