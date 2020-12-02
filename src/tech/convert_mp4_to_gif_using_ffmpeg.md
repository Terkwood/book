# Convert MP4 to GIF using ffmpeg

We captured an mp4 and wanted to convert it to an animated gif, but without the complex invocations below, it will look grainy!

```sh
ffmpeg my-video.mp4 -filter_complex "[0:v] palettegen" /tmp/palette.png
ffmpeg -i my-video.mp4 -i /tmp/palette.png  -filter_complex "[0:v][1:v] paletteuse" my-video.gif
```
