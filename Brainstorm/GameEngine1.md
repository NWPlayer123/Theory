Trying to mishmash together all the smart ideas I've seen from other engines (NintendoWare, Unity, Unreal, Custom)
* Separate folders into approachable tasks, e.g. Effect, Font, Layout, Map (Object Placement), Message/Text, Model, Sound, System, etc
* Nintendo had the smart idea of making "pack" archive blobs that it loads into memory for frequently used data, don't keep opening files
* Programming-wise, Nintendo split tasks into libraries
    * Layout/lyt, deals entirely with 2D data, has layout info files, one for placement, another for animation, and then timg has actual textures
    * Models/g3d/whatever else they call it, 3D data, Model/ sometimes has files with extra textures it just maps onto other models for whatever reason, e.g. Splatoon 2 does this for the Miiverse 2.0 posting thing
Gonna need to add
    * Font, because for some reason kerning and shit is intensive enough that it needs its own library
    * Sound/snd, for audio processing, output to DSP whatever, render effects, decode if need be (ogg, etc)
    * Thanks for leaving your symbols in Kirby Rainbow Curse btw
* Probably add my own libraries for ease of use, Utility, Maths, System, Controls/Controller, I'll decide name later
* Event Queue, make it easier to script out stuff
* Some overlay for graphics to make it portable, e.g. RenderModel -> platform specific low-level graphics pipe
* Object Oriented, obviously, probably separate into sections, Layout, 
* Decide if I wanna just make it a whole scene tree, MainProgram -> Rendering, File I/O, Controller I/O, 
