**STALKER-X** is a camera module for LÖVE. It provides basic functionalities that a camera should have and is inspired by 
[hump.camera](http://hump.readthedocs.io/en/latest/camera.html) and [FlxCamera](http://haxeflixel.com/demos/FlxCamera/). The goal
is to provide enough functions that building something like [in this video](https://www.youtube.com/watch?v=aAKwZt3aXQM)
becomes as easy as possible.

# Contents

<br>

# Quick Start

Place the `Camera.lua` file inside your project and require it:

```lua
Camera = require 'Camera'
```

<br>

## Creating a camera object

```lua
function love.load()
    camera = Camera()
end

function love.update(dt)
    camera:update(dt)
end

function love.draw()
    camera:attach()
    -- Draw your game here
    camera:detach()
    camera:draw() -- Call this here if you're using camera:fade, camera:flash or debug drawing the deadzone
end
```

You can create multiple camera objects if needed, even though most of the time you can get away with just a single global one.

<br>

## Following a target

The main feature of this library is the ability to follow a target. We can do that in a basic way like this:

```lua
function love.update(dt)
    camera:update(dt)
    camera:follow(player.x, player.y)
end
```

And that would look like this:

<br>

## Follow lerp and lead

We can change how sticky or how ahead of the target the camera is by changing the camera's follow lerp and lead variables:

```lua
function love.load()
    camera = Camera()
    camera:setFollowLerp(0.2)
end
```

And that would look like this:

```lua
function love.load()
    camera = Camera()
    camera:setFollowLead(5)
end
```

And that would look like this:

<br>

## Deadzones

Different deadzones define different areas in which the camera will or will not follow the target. 
This can be useful to create all sorts of different behaviors like the some of the ones outlined in 
[this article](https://www.gamasutra.com/blogs/ItayKeren/20150511/243083/Scroll_Back_The_Theory_and_Practice_of_Cameras_in_SideScrollers.php).

### LOCKON

This is the default deadzone.

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('LOCKON')
end
```

<br>

### PLATFORMER

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('PLATFORMER')
end
```

<br>

### TOPDOWN

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('TOPDOWN')
end
```

### TOPDOWN_TIGHT

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('TOPDOWN_TIGHT')
end
```

### SCREEN_BY_SCREEN

In this one the camera will move whenever the target reaches the edges of the screen in a screen by screen basis.
Because of this we need to define the width and height of our screen, which can be seen below as the 3rd and 4rd arguments
to the `Camera` call. In most cases where the external screen size matches the internal screen size this will be done
automatically, but in some cases you might need to define it yourself. 

For instance, if you're doing a pixel art game which has an internal resolution of `360x270`, in which you draw the entire
game to a canvas and then draw the canvas scaled by a factor in the end to fit the final screen size, you'd want the camera's
internal width/height to be the base `360x270`, and not the final `1440x1080` in case of 4x scale factor.

```lua
function love.load()
    camera = Camera(400, 300, 800, 600)
    camera:setFollowStyle('SCREEN_BY_SCREEN')
end
```

### NO_DEADZONE

Without a deadzone the target will just be followed directly and without lerping or leading being applied.

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('NO_DEADZONE')
end
```

<br>

## Shake

```lua
function love.keypressed(key)
    if key == 's' then
        camera:shake(8, 1, 60)
    end
end
```

In this example the camera will shake with intensity 8 and for the duration of 1 second with a frequency of 60Hz.
The camera implementation is based on [this tutorial](https://jonny.morrill.me/en/blog/gamedev-how-to-implement-a-camera-shake-effect/)
which provides a nice additional `frequency` parameter. Higher frequency means jerkier motion, and lower frequency means
smoother motion.

<br>

## Flash

```lua
function love.draw()
    camera:attach()
    -- ...
    camera:detach()
    camera:draw() -- Must call this to use camera:flash!
end

function love.keypressed(key)
    if key == 'f' then
        camera:flash(0.2, {0, 0, 0, 255})
    end
end
```

The example above will fill the screen with the black color for 0.2 seconds, which looks like this:

<br>

## Fade

```lua
function love.draw()
    camera:attach()
    -- ...
    camera:detach()
    camera:draw() -- Must call this to use camera:fade!
end

function love.keypressed(key)
    if key == 'f' then
        camera:flash(1, {0, 0, 0, 255})
    end
    
    if key == 'g' then
        camera:flash(1, {0, 0, 0, 0})
    end
end
```

In the example above, when `f` is pressed the screen will be gradually filled over 1 second with the black color 
and then it will remain covered. If `g` is pressed after that then the screen will gradually go back to normal over 1 second.
The default color that covers the screen initially is `{0, 0, 0, 0}`. 

<br>
