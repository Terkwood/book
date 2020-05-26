# ffmpeg

🤷‍♂

## crop video

You can [crop video](https://video.stackexchange.com/questions/4563/how-can-i-crop-a-video-with-ffmpeg) using ffmpeg.

```sh
ffmpeg -i in.mp4 -filter:v "crop=out_w:out_h:x:y" out.mp4
```

e.g. crop a 1080x800 section starting at 0,1000

```sh
ffmpeg -i in.mp4 -filter:v "crop=1080:600:0:900" out.mp4
```

## Shrink Video

You can [shrink video](https://unix.stackexchange.com/questions/28803/how-can-i-reduce-a-videos-size-with-ffmpeg/38380#38380).

```sh
 ffmpeg -i in_big.mp4 -vcodec libx265 -crf 24 out_small.mp4
```

