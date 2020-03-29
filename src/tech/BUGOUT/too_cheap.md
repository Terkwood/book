# Too Cheap for Pro Tier

It's pretty obvious to anyone that spends 30 minutes in the Kafka literature, that a production deployment of Kafka involves at least three machines.  I was running a single node, because my average user load was zero.  What's more, I was paying for AWS compute time using a personal credit card, so I've never been exactly fired up about running an individual host that comes anywhere near [the specs recommended by the vendor](https://docs.confluent.io/current/kafka/deployment.html).

Some additional reading turned up that [6GB of RAM is a reasonable minimum](https://www.infoq.com/articles/apache-kafka-best-practices-to-optimize-your-deployment/) for any single host:

> In most cases, Kafka can run optimally with 6 GB of RAM for heap space. For especially heavy production loads, use machines with 32 GB or more. Extra RAM will be used to bolster OS page cache and improve client throughput. While Kafka can run with less RAM, its ability to handle load is hampered when less memory is available.

Ever the miser, I settled on running a `t3.medium` compute host for my degenerate Kafka cluster with its single, lonely node.  4GB of RAM certainly worked well enough for my theoretically scalable backend -- at least as long as it didn't become popular.

Anyway, who cares about vendor reccomended minima or a realistic-looking deployment?  I was liberated from corporate hierarchy, profit motive, etc, and was ready to experience the raw freedom that I'd always dreamed about as a junior developer.  I was ready to _play with toys_.

The only problem was that running even a `t3.medium` around the clock [cost about $1/day](https://www.ec2instances.info/?filter=t3&cost_duration=daily).

🤑 That's expensive! 🤑

## Turning Kafka On and Off with Rust + AWS 

The windmills were ready to tilt.

[I spent a non-trivial amount of effort](https://github.com/Terkwood/BUGOUT/issues/75) using [rusoto](https://github.com/rusoto/rusoto) to control the startup and shutdown of my "very expensive" `t3.medium` instance.  Whenever the system detects that there are no users connected, a timer starts, and after a few minutes, [it shuts down](https://github.com/Terkwood/BUGOUT/tree/unstable/reaper) the `t3.medium` running Kafka, and all of the memory-hogging Kafka Streams apps supporting gameplay.

Whenever someone comes back to the website and wants to play a game, the system loyally [boots the Kafka host](https://github.com/Terkwood/BUGOUT/tree/unstable/bugle).

And that's great.  I can still afford to eat.  But the user experience is... abysmal.  It takes about 90 seconds for the EC2 instance and its various docker containers to start up.  (81 seconds longer than eternity!)

## Starting on the Micro-Stack

Enough was enough.  I had no boss.  I had no users.  I had no chains.  My self-respect was questionable.

I would re-implement the backend [with redis streams](https://dev.to/pdambrauskas/event-sourcing-with-redis-45ha), so that my board could be only 24/7, using leftover CPU and RAM on the `t3.micro` instance that I was already paying for.

Why not?  Redis is awesome!  And by choosing `rust` for this portion of the impl, I knew I could keep my memory and CPU footprint low.

[The effort is ongoing](https://github.com/Terkwood/BUGOUT/issues/174). It will eventually be completed in the way that one completes a crocheted scarf.
