FluxEngine
==========

What?
-----

FluxEngine is a very cheap USB floppy disk interface capable of reading (and eventually, writing) exotic non-PC floppy disk formats. It (should, this bit's not done yet) allow you to use a conventional PC drive to accept Amiga disks, CLV Macintosh disks, bizarre 128-sector CP/M disks, and other weird and bizarre formats.

**Big warning.** Right now it is a hacked together prototype. It is not ready to use. Unless you eat and breathe embedded systems code and were born with a soldering iron in your mouth (hopefully, turned off) then this is not for you. If you were... please, give it a try!

How?
----

### Introduction

The system is based around a [Cypress PSoC5LP CY8CKIT-059 development
board](http://www.cypress.com/documentation/development-kitsboards/cy8ckit-059-psoc-5lp-prototyping-kit-onboard-programmer-and),
which is a decently fast ARM core wrapped around a CLDC/FPGA soft logic
device. You can [get one for about
$15](https://www.mouser.com/ProductDetail/Cypress-Semiconductor/CY8CKIT-059?qs=sGAEpiMZZMuo%252bmZx5g6tFKhundMNZurhvz2tw2jO%2fk8%3d).

You need no extra components --- my current prototype (pictured above) has a
17-way header soldered to one side of the board; it then pushes directly onto
the floppy drive connector. Then you plug it into your PC via USB and you're
ready to go.

(Disclaimer: you do, however, need to _remove_ four components from the
board. I'm looking into moving the pinout to avoid this.)

### Bill of materials

So you've decide to go against my advice and attempt to build on of these
suckers. Well, good luck, that's all I can say.

Here's the physical stuff you need.

  - one (1) CY8CKIT-059 development board. See above. If your soldering is
    like mine, you may potentially need more, but _I_ managed to construct the
    thing without frying it so it's not too hard.

  - one (1) standard PC floppy disk drive. You'll have to search around as
    they're increasingly hard to find. The FluxEngine should work with any
    standard 3.5" drive. It's theoretically capable of supporting 5.25"
    drives too but I'd need to modify the firmware timings. If you want this,
    [get in touch](https://github.com/davidgiven/fluxengine/issues/new).

  - some way of connecting the board to your drive. My prototype above uses a
    set of headers to let me attach the board directly on the back of the
    drive; this works fine, but the geometry's kind of awkward as part of the
    board covered the power socket and I had to modify it. (Which is why the
    programmer is hanging off the back.) I'd probably recommend soldering on
    pins instead, and using a traditional floppy cable. That'd let you attach
    two drives, too (although this is currently unsupported in the firmware;
    if you want this, [get in
    touch](https://github.com/davidgiven/fluxengine/issues/new).

  - decent soldering iron skills and an iron with a fine tip --- the pads on
    the board are very small.

  - a Windows machine to run the Cypress SDK on. (The FluxEngine client
    software itself will run on Linux, Windows, and probably OSX, but you
    have to build the firmware on Windows.)

  - optional: a floppy drive power cable. You can cut this in half, soldering
    the raw end to the FluxEngine board, and power the drive off USB --- very
    convenient. This only works for drives which consume less than 500mA.
    _Check before trying_ (5.25" drives need not apply here). Otherwise
    you'll need an actual power supply.

### Assembly instructions

  1. Very carefully remove C7, C9, C12 and C13 from the top of the board.
     It's not hard; use tweezers or fine needle-nosed pliers to lift the
     component, apply heat to one end, wait for a few seconds and as the heat
     soaks through the solder will soften and it'll come away. Be careful;
     they're incredibly small. (I'm looking at changing the pinout to avoid
     needing this step, which is first so that if you ruin your board you don't waste time.)

  2. **If you're using a header:** solder your 17-way header to the
     **bottom** of the board, from 0.2 to 3.0 inclusive. (It has to be the
     bottom because there are components that stick out on the other side and
     the bottom needs to go flush against the drive.)

  3. **If you're using pins:** solder you 17-way pins to either side of the
     board, from 0.2 to 3.0 inclusive.

And you're done!

### Building the firmware

On your Windows machine, [install the Cypress SDK and CY8CKIT-059
BSP](http://www.cypress.com/documentation/development-kitsboards/cy8ckit-059-psoc-5lp-prototyping-kit-onboard-programmer-and).
This is a frustratingly long process and there are a lot of moving parts; you
may need to register. You want the file from the above site marked 'Download
CY8CKIT-059 Kit Setup (Kit Design Files, Creator, Programmer, Documentation,
Examples)'. I'm not linking to it in case the URL changes when they update
it.

Once this is done, I'd strongly recommend working through the initial
tutorial and making the LED on your board flash. The FluxEngine firmware
isn't nearly ready for fire-and-forget use and you'll probably need to tweak
it. (If you do, please [get in
touch](https://github.com/davidgiven/fluxengine/issues/new)).

When you're ready, open the `FluxEngine.cydsn/FluxEngine.cywrk` workspace,
pick 'Program' from the menu, and the firmware should compile and be
programmed onto your board.

### Building the client

The client software is where the intelligence, such as it is, is. It's pretty
generic libusb stuff and should build and run on Windows, Linux and probably
OSX as well, although on Windows I've only ever used it with Cygwin. You'll
need the `sqlite3`, `meson` and `ninja` packages (which should be easy to
come by in your distro). Just do `make` and it should build.

If it doesn't build, please [get in
touch](https://github.com/davidgiven/fluxengine/issues/new).

### Using it

So you have client software, firmware, and hardware all ready. What next?

  1. Attach the FluxEngine to your floppy disk drive. Pin 

The sampling system is dumb as rocks.

There's an 8-bit counter attached to an 12MHz clock. This is used to measure
the interval between pulses. If the timer overflows, we pretend it's a pulse
(this very rarely happens in real life).

An HD floppy has a nominal clock of 500kHz, so we use a sample clock of 12MHz
(every 83ns). This means that our 500kHz pulses will have an interval of 24
(and a DD disk with a 250kHz nominal clock has an interval of 48). This gives
us more than enough resolution. If no pulse comes in, then we sample on
rollover at 21us.

(The clock needs to be absolutely rock solid or we get jitter which makes the
data difficult to analyse, so 12 was chosen to be derivable from the
ultra-accurate USB clock.)

VERY IMPORTANT:

Some of the pins on the PSoC have integrated capacitors, which will play
havoc with your data! These are C7, C9, C12 and C13. If you don't, your
floppy drive will be almost unusable (the motor will run but you won't see
any data). They're easy enough with some tweezers and a steady hand with a
soldering iron.

Some useful numbers:

  - nominal rotation speed is 300 rpm, or 5Hz. The period is 200ms.
  - a pulse is 150ns to 800ns long.
  - a 12MHz tick is 83ns.
  - MFM HD encoding uses a clock of 500kHz. This makes each recording cell 2us,
    or 24 ticks. For DD it's 4us and 48 ticks.
  - a short transition is one cell (2us == 24 ticks). A medium is a cell and
    a half (3us == 36 ticks). A long is two cells (4us == 48 ticks). Double
    that for DD.
  - pulses are detected with +/- 350ns error for HD and 700ns for DD. That's
    4 ticks and 8 ticks. That seems to be about what we're seeing.
  - in real life, start going astray after about 128 ticks == 10us. If you
    don't write anything, you read back random noise.
  
Useful links:

http://www.hermannseib.com/documents/floppy.pdf

https://hxc2001.com/download/datasheet/floppy/thirdparty/Teac/TEAC%20FD-05HF-8830.pdf
