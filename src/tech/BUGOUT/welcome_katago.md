# Welcome KataGo

After months of relatively steady operation, I came to the sobering conclusion that most of my close friends were busy with their own lives, and playing a synchronous session of GO with them, often across international time zones, was an exercise in scheduling prowess that surpasses even the most skilled project manager.

So I began to wonder about playing against an [AlphaZero-like](https://en.wikipedia.org/wiki/AlphaZero) AI through my browser.

If I can't play against my friends...

...why not _BUILD MY OWN FRIEND?_

## Kata Meets Nano

Thus was born the effort to integrate [KataGo](https://github.com/lightvector/KataGo), a community implementation of cutting edge Go AI that's gotten a nice little boost thanks to [hardware sponsorship from Jane Street](https://blog.janestreet.com/accelerating-self-play-learning-in-go/).

In fact, [you can already play against KataGo online](https://online-go.com/player/592684/).

That's fine. I like building things on my own -- or in this case, integrating other people's things on my own.  So I purchased an [NVIDIA Jetson Nano System-on-a-Chip](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-nano-developer-kit/) and set about [linking it to my existing infrastructure](https://github.com/Terkwood/BUGOUT/issues/67).

The effort proceeded smoothly.  Soon I was suffering regular blows to my pride, delivered easily by a 128-core GPU that can subside on a mere 5W of power.  NVIDIA's pre-installed CUDA libs made compilation of KataGo relatively easy.  Hooking up the tiny SoC in my office to my dirt-cheap AWS `t3.micro` felt pleasantly conservative.

![Nvidia Jetson Nano Unboxed](https://user-images.githubusercontent.com/38859656/77859879-e611f580-71d9-11ea-898a-e2f928605cac.jpeg)
