**STALKER-X** is a camera module for LÖVE. It provides basic functionalities that a camera should have and is inspired by 
[hump.camera](http://hump.readthedocs.io/en/latest/camera.html) and [FlxCamera](http://haxeflixel.com/demos/FlxCamera/). The goal
is to provide enough functions that building something like [in this video](https://www.youtube.com/watch?v=aAKwZt3aXQM)
becomes as easy as possible.

# Contents

* [Quick Start](#quick-start)
   * [Creating a camera object](#creating-a-camera-object)
   * [Following a target](#following-a-target)
   * [Follow lerp and lead](#follow-lerp-and-lead)
   * [Deadzones](#deadzones)
      * [LOCKON](#lockon)
      * [PLATFORMER](#platformer)
      * [TOPDOWN](#topdown)
      * [TOPDOWN_TIGHT](#topdown_tight)
      * [SCREEN_BY_SCREEN](#screen_by_screen)
      * [NO_DEADZONE](#no_deadzone)
      * [Custom Deadzones](#custom-deadzones)
   * [Shake](#shake)
   * [Flash](#flash)
   * [Fade](#fade)
* [Tips](#tips)
   * [Pixel Camera](#pixel-camera)
   * [Fixed Timestep](#fixed-timestep)
* [Documentation](#documentation)
   * [Camera](#camerax-y-w-h-scale-rotation)
   * [update](#updatedt)
   * [draw](#draw)
   * [attach](#attach)
   * [detach](#detach)
   * [x, y](#x-y)
   * [scale](#scale)
   * [rotation](#rotation)
   * [toWorldCoords](#toworldcoordsx-y)
   * [toCameraCoords](#tocameracoordsx-y)
   * [getMousePosition](#getmouseposition)
   * [shake](#shakeintensity-duration-frequency-axes)
   * [flash](#flashduration-color)
   * [fade](#fadeduration-color)
   * [follow](#followx-y)
   * [setFollowStyle](#setfollowstylefollow_style)
   * [setDeadzone](#setdeadzonex-y-w-h)
   * [draw_deadzone](#draw_deadzone)
   * [setFollowLerp](#setfollowlerpx-y)
   * [setFollowLead](#setfollowleadx-y)
   * [setBounds](#setboundsx-y-w-h)

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


<p align="center">
  <img src="https://i.imgur.com/CmGYXxW.gif"/>
</p>

<br>

## Follow lerp and lead

We can change how sticky or how ahead of the target the camera is by changing its lerp and lead variables. 
Lerp is value that goes from 0 to 1. Closer to 0 means less sticky following, while closer to 1 means stickier following:

```lua
function love.load()
    camera = Camera()
    camera:setFollowLerp(0.2)
end
```

And that would look like this:

<p align="center">
  <img src="https://i.imgur.com/qtdTzU6.gif"/>
</p>

Lead is a value that goes from 0 to infinity. Closer to 0 means no look-ahead, while higher values will move the camera
in the direction of the target's movement more. In practice good lead values will range from 2 to 10.

```lua
function love.load()
    camera = Camera()
    camera:setFollowLerp(0.2)
    camera:setFollowLead(10)
end
```

<p align="center">
  <img src="https://i.imgur.com/ZcOZ6n6.gif"/>
</p>

<br>

## Deadzones

Different deadzones define different areas in which the camera will or will not follow the target. 
This can be useful to create all sorts of different behaviors like the some of the ones outlined in 
[this article](https://www.gamasutra.com/blogs/ItayKeren/20150511/243083/Scroll_Back_The_Theory_and_Practice_of_Cameras_in_SideScrollers.php). All the examples below use a lerp
value of 0.2 and a lead value of 0.

### LOCKON

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('LOCKON')
end
```

<p align="center">
  <img src="https://i.imgur.com/9NCOySe.gif"/>
</p>

### PLATFORMER

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('PLATFORMER')
end
```

<p align="center">
  <img src="https://i.imgur.com/eNny8VU.gif"/>
</p>

### TOPDOWN

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('TOPDOWN')
end
```

<p align="center">
  <img src="https://i.imgur.com/eA57zFd.gif"/>
</p>

### TOPDOWN_TIGHT

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('TOPDOWN_TIGHT')
end
```

<p align="center">
  <img src="https://i.imgur.com/FqiTqOM.gif"/>
</p>

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
    camera = Camera(200, 150, 400, 300)
    camera:setFollowStyle('SCREEN_BY_SCREEN')
end
```

<p align="center">
  <img src="https://i.imgur.com/SN1S0Jo.gif"/>
</p>

### NO_DEADZONE

Without a deadzone the target will just be followed directly and without lerping or leading being applied.
If the lerp value is 1 and the lead value is 0 (the default values for both of those) then the camera will act
just like in the `NO_DEADZONE` mode, even though the default mode is `LOCKON`.

```lua
function love.load()
    camera = Camera()
    camera:setFollowStyle('NO_DEADZONE')
end
```

<p align="center">
  <img src="https://i.imgur.com/TBQE88l.gif"/>
</p>

### Custom Deadzones

Custom deadzones can be set with the `:setDeadzone(x, y, w, h)` call. Deadzones are set in camera coordinates, 
with the top-left being `0, 0` and the bottom-right being `camera.w, camera.h`. So the following call:

```lua
function love.load()
    local w, h = 400, 300
    camera = Camera(w/2, h/2, w, h)
    camera:setDeadzone(40, h/2 - 40, w - 80, 80)
end
```

Will result in this:

<p align="center">
  <img src="https://i.imgur.com/rDpdvRI.gif"/>
</p>

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

<p align="center">
  <img src="https://i.imgur.com/0GaczA3.gif"/>
</p>

Note that if you have a target locked and you have `NO_DEADZONE` or a lerp of 1 set, then a screen shake won't happen
since the camera will be locked tightly to the target.

<br>

## Flash

This is a good effect for when the player gets hit, lightning strikes, or similar events.

```lua
function love.draw()
    camera:attach()
    -- ...
    camera:detach()
    camera:draw() -- Must call this to use camera:flash!
end

function love.keypressed(key)
    if key == 'f' then
        camera:flash(0.05, {0, 0, 0, 1})
    end
end
```

The example above will fill the screen with the black color for 0.05 seconds, which looks like this:

<p align="center">
  <img src="https://i.imgur.com/UkhiyzE.gif"/>
</p>

<br>

## Fade

This is a good effect for transitions between levels.

```lua
function love.draw()
    camera:attach()
    -- ...
    camera:detach()
    camera:draw() -- Must call this to use camera:fade!
end

function love.keypressed(key)
    if key == 'f' then
        camera:fade(1, {0, 0, 0, 1})
    end
    
    if key == 'g' then
        camera:fade(1, {0, 0, 0, 0})
    end
end
```

In the example above, when `f` is pressed the screen will be gradually filled over 1 second with the black color 
and then it will remain covered. If `g` is pressed after that then the screen will gradually go back to normal over 1 second.
The default color that covers the screen initially is `{0, 0, 0, 0}`. 

<p align="center">
  <img src="https://i.imgur.com/o3r2noG.gif"/>
</p>

<br>

# Tips

## Pixel Camera

All the gifs above were created with what I call a pixel art setup. In that everything is drawn to a canvas at a base resolution
and then that canvas is scaled to the final screen using the `nearest` filter mode. This is how a chunky pixel look can be achieved and it's generally how pixel art is scaled in games. The advantages of this method is that you only have to care
about a single resolution and then everything else takes care of itself. The way this setup looks like in LÖVE code could go
something like this:

```lua
function love.load()
    love.graphics.setDefaultFilter('nearest', 'nearest') -- scale everything with nearest neighbor
    canvas = love.graphics.newCanvas(400, 300)
end

function love.draw()
    love.graphics.setCanvas(canvas)
    love.graphics.clear()
    -- draw the game here
    love.graphics.setCanvas()
    
    -- Draw the 400x300 canvas scaled by 2 to a 800x600 screen
    love.graphics.setColor(1, 1, 1, 1)
    love.graphics.setBlendMode('alpha', 'premultiplied')
    love.graphics.draw(canvas, 0, 0, 0, 2, 2)
    love.graphics.setBlendMode('alpha')
end
```

All the gifs above followed this code. It's a base resolution of `400x300` being drawn at a scale of 2 to a `800x600` screen.

Now, this relates to the camera in that to make the camera work with this setup we need to tell it what's the base resolution
we're using. In this case it's `400x300` and so we can create the camera object specifying these values:

```lua
function love.load()
    camera = Camera(200, 150, 400, 300)
    ...
end
```

The third and fourth arguments of the `Camera` call are for the internal width and height of the camera, and in this case
they should match the base resolution. If those arguments are omitted then it will default to whatever value is returned
by the `love.graphics.getWidth` and `love.graphics.getHeight` calls. In a pixel setup like this omitting those values is 
problematic because then the camera would assume an internal resolution of `800x600` which would make everything not work properly.

<br>

## Fixed Timestep

If you're using a variable timestep you might notice a jerky motion when the camera tries to follow a target tightly.
This can be fixed by decreasing the lerp value, or more cleanly by using a fixed timestep setup. The code below is based
on the "Free the Physics" section of [this article](https://gafferongames.com/post/fix_your_timestep/).

```lua
-- LÖVE 0.10.2 fixed timestep loop, Lua version
function love.run()
    if love.math then love.math.setRandomSeed(os.time()) end
    if love.load then love.load(arg) end
    if love.timer then love.timer.step() end

    local dt = 0
    local fixed_dt = 1/60
    local accumulator = 0

    while true do
        if love.event then
            love.event.pump()
            for name, a, b, c, d, e, f in love.event.poll() do
                if name == 'quit' then
                    if not love.quit or not love.quit() then
                        return a
                    end
                end
                love.handlers[name](a, b, c, d, e, f)
            end
        end

        if love.timer then
            love.timer.step()
            dt = love.timer.getDelta()
        end

        accumulator = accumulator + dt
        while accumulator >= fixed_dt do
            if love.update then love.update(fixed_dt) end
            accumulator = accumulator - fixed_dt
        end

        if love.graphics and love.graphics.isActive() then
            love.graphics.clear(love.graphics.getBackgroundColor())
            love.graphics.origin()
            if love.draw then love.draw() end
            love.graphics.present()
        end

        if love.timer then love.timer.sleep(0.0001) end
    end
end
```

```moonscript
-- LÖVE 0.10.2 fixed timestep loop, MoonScript version
love.run = () ->
  if love.math then love.math.setRandomSeed(os.time())
  if love.load then love.load(arg)
  if love.timer then love.timer.step()

  dt = 0
  fixed_dt = 1/60
  accumulator = 0

  while true
    if love.event
      love.event.pump()
      for name, a, b, c, d, e, f in love.event.poll() do
        if name == "quit"
          if not love.quit or not love.quit()
            return a
        love.handlers[name](a, b, c, d, e, f)

    if love.timer
      love.timer.step()
      dt = love.timer.getDelta()

    accumulator += dt
    while accumulator >= fixed_dt do
      if love.update then love.update(fixed_dt)
      accumulator -= fixed_dt

    if love.graphics and love.graphics.isActive()
      love.graphics.clear(love.graphics.getBackgroundColor())
      love.graphics.origin()
      if love.draw then love.draw()
      love.graphics.present()

    if love.timer then love.timer.sleep(0.0001)
```

<br>

# DOCUMENTATION

#### `Camera(x, y, w, h, scale, rotation)`

Creates a new Camera.

```lua
camera = Camera()
```

Arguments:

* `x=w/2` `(number)` - The camera's x position. Defaults to `w/2`
* `y=h/2` `(number)` - The camera's y position. Defaults to `h/2`
* `w=love.graphics.getWidth()` `(number)` - The camera's width. Defaults to `love.graphics.getWidth()`
* `h=love.graphics.getHeight()` `(number)` - The camera's height. Defaults to `love.graphics.getHeight()`
* `scale=1` `(number)` - The camera's scale. Defaults to `1`
* `rotation=0` `(number)` - The camera's rotation. Defaults to `0`

Returns:

* `Camera` `(table)` - the Camera object

---

#### `:update(dt)`

Updates the camera.

```lua
camera:update(dt)
```

Arguments:

* `dt` `(number)` - The time step delta

---

#### `:draw()`

Draws the camera, drawing the deadzone if `draw_deadzone` is `true` and also drawing the `flash` and `fade` effects.

```lua
camera:draw()
```

---

#### `:attach()`

Attaches the camera, making all following draw operations be affected by the camera's translation, scale and rotation transformations.

```lua
camera:attach()
-- draw the game here
camera:detach()
```

---

#### `:detach()`

Detaches the camera, returning the transformation stack back to normal.

```lua
camera:attach()
-- draw the game here
camera:detach()
```

---

#### `.x, .y`

The camera's position. This is the center of the camera and not its top-left position. This can be changed directly although
if you're using the `follow` function then changing this directly might result in bugs.

```lua
camera.x, camera.y = 0, 0
```

---

#### `.scale`

The camera's scale/zoom. 

```lua
camera.scale = 2
```

---

#### `.rotation`

The camera's rotation.

```lua
camera.rotation = math.pi/8
```

---

#### `:toWorldCoords(x, y)`

The same as [hump.camera:worldCoords](http://hump.readthedocs.io/en/latest/camera.html#camera:worldCoords). This takes in 
a position in camera coordinates and translates it to world coordinates. An example of this is taking the position of the
mouse and seeing where it is in the world.

```lua
mx, my = camera:toWorldCoords(love.mouse.getPosition())
```

Arguments:

* `x` `(number)` - The x position in camera coordinates
* `y` `(number)` - The y position in camera coordinates

Returns:

* `x` `(number)` - The x position in world coordinates
* `y` `(number)` - The y position in world coordinates

---

#### `:toCameraCoords(x, y)`

The same as [hump.camera:cameraCoords](http://hump.readthedocs.io/en/latest/camera.html#camera:cameraCoords). This takes in 
a position in world coordinates and translates it to camera coordinates. An example of this is taking the position of the
player and 

```lua
player_x, player_y = camera:toCameraCoords(player.x, player.y)
love.graphics.line(player_x, player_y, love.mouse.getPosition())
```

Arguments:

* `x` `(number)` - The x position in world coordinates
* `y` `(number)` - The y position in world coordinates

Returns:

* `x` `(number)` - The x position in camera coordinates
* `y` `(number)` - The y position in camera coordinates

---

#### `:getMousePosition()`

Gets the position of the mouse in world coordinates. This position can also be accessed directly through `.mx, .my`.

```lua
mx, my = camera:getMousePosition()
mx, my = camera.mx, camera.my
```

Returns:

* `x` `(number)` - The x position of the mouse in world coordinates
* `y` `(number)` - The y position of the mouse in world coordinates

---

#### `:shake(intensity, duration, frequency, axes)`

Shakes the screen with intensity for a certain duration.

```lua
camera:shake(8, 1, 60, 'X')
```

Arguments:

* `intensity` `(number)` - The intensity of the shake in pixels. This will be decreased along the duration of the shake.
* `duration=1` `(number)` - The duration of the shake in seconds. Defaults to `1`
* `frequency=60` `(number)` - The frequency of the shake. Higher = jerkier, lower = smoother. Defaults to `60`
* `axes='XY'` `(string)` - The axes of the shake. Can be `'X'` for horizontal, `'Y'` for vertical or `'XY'` for both. Defaults to `'XY'`

---

#### `:flash(duration, color)`

Fills the screen up with a color for a certain duration.

```lua
camera:flash(0.05, {0, 0, 0, 1})
```

Arguments:

* `duration` `(number)` - The duration of the flash in seconds
* `color={0, 0, 0, 1}` `(table[number])` - The color of the flash. Defaults to `{0, 0, 0, 1}`

---

#### `:fade(duration, color, action)`

Slowly fills up the screen with a color along the duration.

```lua
camera:fade(1, {0, 0, 0, 1}, function() print(1) end)
```

Arguments: 

* `duration` `(number)` - The duration of the fade in seconds
* `color` `(table[number])` - The target color of the fade
* `action` `function` - An optional action that is run when the fade ends

---

#### `:follow(x, y)`

Follow the target according to the follow style and lerp, lead values.

```lua
camera:follow(player.x, player.y)
```

Arguments:

* `x` `(number)` - The x position of the target in world coordinates
* `y` `(number)` - The y position of the target in world coordinates

---

#### `:setFollowStyle(follow_style)`

Sets the follow style to be used by `camera:follow`. Possible values are `'LOCKON'`, `'PLATFORMER'`, `'TOPDOWN'`, `'TOPDOWN_TIGHT'`, `'SCREEN_BY_SCREEN'` and `'NO_DEADZONE'`. This can also be changed directly through `.follow_style`.

```lua
camera:setFollowStyle('LOCKON')
camera.follow_style = 'LOCKON'
```

Arguments:

* `follow_style` `(string)` - The follow style to be used

---

#### `:setDeadzone(x, y, w, h)`

Sets the deadzone directly. The follow style must be set to `nil` for this to work.

```lua
camera:setDeadzone(0, 0, w, h)
```

Arguments:

* `x` `(number)` - The top-left x position of the deadzone in camera coordinates
* `y` `(number)` - The top-left y position of the deadzone in camera coordinates
* `w` `(number)` - The width of the deadzone
* `h` `(number)` - The height of the deadzone

---

#### `.draw_deadzone`

Draws the deadzone if set to true. `camera:draw()` must be called outside the `camera:attach/detach` block for it to work.

```lua
camera.draw_deadzone = true
```

---

#### `:setFollowLerp(x, y)`

Sets the lerp value. This can be accessed directly through `.follow_lerp_x` and `.follow_lerp_y`.

```lua
camera:setFollowLerp(0.2)
camera.follow_lerp_x = 0.2
camera.follow_lerp_y = 0.2
```

Arguments:

* `x` `(number)` - The x lerp value
* `y=x` `(number)` - The y lerp value. Defaults to the `x` value

---

#### `:setFollowLead(x, y)`

Sets the lead value. This can be accessed directly through `.follow_lead_x` and `.follow_lead_y`.

```lua
camera:setFollowLead(10)
camera.follow_lead_x = 10
camera.follow_lead_y = 10
```

Arguments:

* `x` `(number)` - The x lead value
* `y=x` `(number)` The y lead value. Defaults to the `x` value

---

#### `:setBounds(x, y, w, h)`

Sets the boundaries of the camera in world coordinates. The camera won't be able to move past those points.

```lua
camera:setBounds(0, 0, 800, 600)
```

Arguments:

* `x` `(number)` - The top-left x position of the boundary
* `y` `(number)` - The top-left y position of the boundary
* `w` `(number)` - The width of the rectangle that defines the boundary
* `h` `(number)` - The height of the rectangle that defines the boundary

---

<br>

# LICENSE

You can do whatever you want with this. See the license at the top of the main file.
