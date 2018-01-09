Framerate is another important aspect cuz it puts the video in video games. The problem, however, is getting a consistent framerate. Most people would agree that framerate is more important than resolution. That being said, it's sometimes impossible to maintain a solid [30|60] FPS, there's bound to be a few dropped frames.

The problem is basically, you have to do game physics calculations, and render an image, and print it to the screen, 60 times per second. So basically, you have 16.6666 milliseconds to do all your calculations and render out a [480|720|1080]p image to keep up.

If you update your game physics after every frame, you run into a problem. If you designed your stuff to run at 30FPS, and it's only updating 24 times (24 fps), you're behind 6 physics updates.

The way developers get around this is something called 'delta time'. Basically, you run physics updates independent of the framerate, so one second in real life is one second worth of updates, even if you're only displaying 17FPS. You have a timer, and every refresh you check time that's passed, do some multiplication (I need to move the model coordinates this much), and then write the new data, which the rendering thread will check next time it comes around.

Modern games have these fancy threads that can let you do multiple things at once. 

* One thread does rendering, feeding all the information to the GPU and rendering it to the screen
* One thread keeps sound up to date (it plays at [32000|44100|48000]Hz per second so keeping it independent of framerate is also important)
* ‎One thread checks controller inputs and relays them to the physics thread so it can update as necessary.
* ‎One thread for physics, updating positions of all the objects, checking collision, doing some fancy cloth physics like BotW or a building crumbling or wind blowing
* ‎One thread can handle network if you implement that, probably want it independent of controller input since it has a lot to do even on its own, check that data coming in is valid, processing it, getting and sending data back
* ‎One thread loads resources from disk or the network, usually in advance, which lets you update frames for a loading screen while it gets models and textures in the background.

Keep in mind these are all running at the same time to make your game work, 30 or 60 times per second.

# Dealing with frame drops
Using the above assumptions, there are multiple ways you can deal with "dropped" frames. If you assume that you can't hit that 16ms window for the update, you have a few options. You can either keep the old frame up (so you see the frame for two frames, it's dropped), and the rendering thread will do another loop, get the updated physics data, and it'll keep up as if you'd hit that 16ms window.

Another thing you can do is have *another* thread running, estimating how long it will take to render a frame, and if it'll take too long, maybe render at a lower resolution? (instead of rendering the 1080p image you need, you render a 480p image just to keep the brain going, look up "persistence of vision").

The problem is that it has to feed the GPU X number of objects and textures. Which is easier, updating 30 models every 16.66ms, or 300?

You can also attempt to guess ahead, if you know you're gonna miss that window, maybe start rendering the frame before you update all the objects, and then prioritize the missed objects the next frame.
