# BUGOUT: Play Go against Strong AI and Human Friends

[BUGOUT](https://github.com/Terkwood/BUGOUT) provides a web interface to [KataGo](https://github.com/lightvector/KataGo), a leading, community-driven AI implementation to the [classic board game, Go](https://en.wikipedia.org/wiki/Go_(game)).  

BUGOUT also features a multiplayer option for playing against human friends, if you're lucky enough to have them.

BUGOUT's web interface is a [derivative](https://github.com/Terkwood/Sabaki) of [Sabaki](https://github.com/SabakiHQ/Sabaki), the much-loved desktop app which is already capable of hooking into KataGo, [Leela Zero](https://zero.sjeng.org/home), and [GNU Go](https://www.gnu.org/software/gnugo/).

## Emotional Backdrop

I started playing GO sometime during high school.  Several of my friends also learned to play.  We would often play chess, for which I had a neat little carrying case with a simple game clock, a rollout board, and sturdy pieces.  I came into a possession of a GO board of standard dimensions, and used plastic pieces that were just heavy enough to feel like they mattered.

We would sometimes meet at a Chinese dive on West 10th Street in Indianapolis, and consume inexpensive but delicious food, and play.

GO felt different than chess.  There was a sense of immense possibility.  There was time to think about the future.  Winning was hard.  Getting your skill up with a same-stage newbie was exhilarating, because you grew together.

Eventually I got my board signed by [VICTOR WOOTEN](TODO) while attending a concert performance given by [The Flecktones](TODO) at the [Indiana Roof Ballroom](TODO):  we had been playing some small rounds prior to the start of the concert.  It must have been 1999.

The music was great, and the signature fueled my fire.

![TODO: image](of the go board signed by wooten)

I was captivated.

In college I dreamt that one day I would wake to the sound of an obstreperous trumpet, having been mired in games and strenuous efforts to build some sort of ultimate strategy.  I dreamt that I woke to a horn and a vision of clear dawn.

The actual time I've spent playing GO in my life is tiny.  Life goes on, and its daily demands slowly obscure our roots.

Still, the aesthetic delight of GO has been enjoyed by millions of individuals over the centuries, and it's this very beauty which has inspired my work over the last nine months.

GO is a game which helps me combat the lassitude of experience and boredom.

GO is not a game which must be won.  It's enough to face a player whose skill exceeds yours, and to learn.  When I play 9x9 against KataGo running on a 5W NVIDIA board, I'm happy if I manage not to lose an embarrassing number of pieces.

_GO keeps me *sharp*_.

## Design of the Project

BUGOUT started out as an excuse to implement something (anything) using [Kafka](TODO).  My previous two professional engagements had barely skirted opportunities to legitimately include a Kafka install:  one was simple enough to rely on HTTP microservices, [another had opted for Kinesis](https://github.com/WW-Digital/reactive-kinesis).

Fresh into successful semi-retirement, I didn't want to bother with finding a use case for my implementation:  I wanted what I couldn't have reasonably argued for in a professional setting... I wanted to play with [Kafka Streams](https://kafka.apache.org/documentation/streams/)!

It turns out that mapping the tiny domain model of a GO game to kafka streams is... easy.  Initially BUGOUT was envisioned as a multiplayer GO board which could easily run in any modern browser and provide an enjoyable, boutique alternative to more popular servers and venues.  I wanted to play again with my old friends, just like when we were kids.  So the project [quickly incorporated a game lobby system](https://github.com/Terkwood/BUGOUT/issues/42) which allowed players to choose between joining a quick 19x19 game with the next person to visit the web page, or creating a nicely-formatted URL (`https://example.com/?join=nXblGBE7erWyocXYYpRN1YOzdD`) for their private game and sharing it with a friend.

![Choosing a Venue](https://user-images.githubusercontent.com/38859656/77851229-e42e3f00-71a5-11ea-8a91-93da91abf87a.png)

![Board Size Options](https://user-images.githubusercontent.com/38859656/77851228-e2647b80-71a5-11ea-9467-cf086d76fb8e.png)

![Turn Order Preference](https://user-images.githubusercontent.com/38859656/77851227-e1cbe500-71a5-11ea-8faa-264996a2257a.png)

![Sharing a Link to a Game](https://user-images.githubusercontent.com/38859656/77851226-e1334e80-71a5-11ea-834f-0c76a0e35080.png)

![Joining a Link to a Game](https://user-images.githubusercontent.com/38859656/77851225-ded0f480-71a5-11ea-92a6-bb23755df74c.png)

Finally, both players can be fairly assigned a side, and the game can begin.

![Your Color](https://user-images.githubusercontent.com/38859656/77851236-e7c1c600-71a5-11ea-94a3-750bbeab74df.png)

![Finally, A Game](https://user-images.githubusercontent.com/38859656/77851232-e55f6c00-71a5-11ea-827e-f0201c6d9f51.png)

It basically worked.  I spent time [making sure the websocket connection between the browser and the BUGOUT gateway server was solid](TODO).  I had to [guarantee that Kafka and all its dependent apps started up in an orderly fashion](TODO).  And I enjoyed writing [functional-ish Kafka Streams code in Kotlin](TODO/representative.bugout.source.link):  from a cognitive perspective, it felt clean and tidy, even if [the topology graphs quickly got out of hand](TODO pic from BUGOUT repo).

## Too Cheap For Pro Gear After All

It's pretty obvious to anyone that spends 30 minutes in the Kafka literature, that a production deployment of Kafka involves at least three machines.  I was running a single node, because my average user load was zero.  What's more, I'm paying for Amazon Compute time using a personal credit card, so I've never been exactly fired up about running an individual host that comes anywhere near [the specs recommended by the vender](https://docs.confluent.io/current/kafka/deployment.html).

Some additional reading turned up that [6GB of RAM is a reasonable minimum](https://www.infoq.com/articles/apache-kafka-best-practices-to-optimize-your-deployment/) for any single host:

> In most cases, Kafka can run optimally with 6 GB of RAM for heap space. For especially heavy production loads, use machines with 32 GB or more. Extra RAM will be used to bolster OS page cache and improve client throughput. While Kafka can run with less RAM, its ability to handle load is hampered when less memory is available.

So, dutifully stingy, I settled on running a `t3.medium` compute host for my degenerate Kafka cluster with its single, lonely node.  Anyway, who cares?  I was liberated from corporate hierarchy, profit motive, etc, and was ready to experience the raw freedom that I'd always dreamed about as a junior developer.  I was ready to play with toys!

The only problem was that running even a `t3.medium` around the clock [cost about $1/day](https://www.ec2instances.info/?filter=t3&cost_duration=daily), which was way too rich for my blood.

## Turning Kafka On and Off with Rust + AWS 

The windmills were ready to tilt.  Fully.

[I spent a non-trivial amount of effort](https://github.com/Terkwood/BUGOUT/issues/75) using [rusoto](https://github.com/rusoto/rusoto) to control the startup and shutdown of my "very expensive" `t3.medium` instance.  Whenever the system detects that there are no users connected, a timer starts, and after a few minutes, [it shuts down](TODO/link/to/reaper) the `t3.medium` running Kafka, and all of the memory-hogging Kafka Streams apps supporting gameplay.

Whenever someone comes back to the website and wants to play a game, the system loyally [boots the Kafka host](TODO/link/bugle).

And that's great.  I can still afford to eat.  But the user experience isn't superb.  It takes about 90 seconds for the EC2 instance and its various docker containers to start up.  

That's about 76 seconds longer than eternity.

![Waiting for the end of time](TODO/image/waiting/startup)

## Starting on the Micro-Stack

Enough was enough.  I had no boss.  I had no users.  I had no chains.  My self-respect was questionable.

I would re-implement the backend [with redis streams and rust](TODO/dev/to/article/link), so that my board could be only 24/7, using leftover CPU and RAM on the `t3.micro` instance that I was already paying for.

Why not?  Redis is awesome!



## Integrate Your Own Friend

But after months of relatively steady operation, I came to the sobering conclusion that most of my close friends were busy with their own lives, and playing a synchronous session of GO with them, often across international time zones, was an exercise in scheduling prowess that surpasses even the most skilled project manager.

So I began to wonder about playing against an [AlphaZero-like](TODO) AI through my browser.

If I can't play against my friends...

...why not _BUILD MY OWN FRIEND?_