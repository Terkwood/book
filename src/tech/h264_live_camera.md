# Raspberry Pi + H264 Live Camera + Web

We found a [really nice project](https://github.com/131/h264-live-player)
which lets you stream video from a Raspberry Pi (model 3 B+, in our case).
We connected the [Camera Module v2](https://www.raspberrypi.org/products/camera-module-v2/) to ours, and had good results.

The project uses a reasonable frame rate to deliver nice results, despite
the Pi's small processing power.

We [created a fork of the project](https://github.com/Terkwood/h264-live-player) which kills the `raspivid` project on disconnect, and simplifies the UI.
