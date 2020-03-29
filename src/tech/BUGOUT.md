# BUGOUT: Play Go against Strong AI and Human Friends

[BUGOUT](https://github.com/Terkwood/BUGOUT) provides a web interface to [KataGo](a/b/c), a leading, community-driven AI implementation to the [classic board game, Go](a/b/c).  

BUGOUT also features a multiplayer option for playing against human friends, if you're lucky enough to have them.

BUGOUT's web interface is a [derivative](path/to/fork) of [Sabaki](path/to), the much-loved desktop app which is capable of hooking directly into KataGo, Leela Zero, and GNU Go.

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

![Choosing a Venue](https://user-images.githubusercontent.com/38859656/77849069-66276380-71b8-11ea-98d9-7e4846161f74.png)

![Board Size Options](TODO)

![Turn order](TODO)

![Copy Link](TODO)

![Joining a Link](TODO)

Finally, both players can be fairly assigned a side, and the game can begin.

![Your Color](TODO)

![Finally, A Game](TODO/finally_a_game.png)

It basically worked.  I spent time [making sure the websocket connection between the browser and the BUGOUT gateway server was solid](TODO).  I had to [guarantee that Kafka and all its dependent apps started up in an orderly fashion](TODO).  And I enjoyed writing [functional-ish Kafka Streams code in Kotlin](TODO representative bugout source link):  from a cognitive perspective, it felt clean and tidy, even if [the topology graphs quickly got out of hand](TODO pic from BUGOUT repo).

## Integrate Your Own Friend

But after months of relatively steady operation, I came to the sobering conclusion that most of my close friends were busy with their own lives, and playing a synchronous session of GO with them, often across international time zones, was an exercise in scheduling prowess that surpasses even the most skilled project manager.

So I began to wonder about playing against an [AlphaZero-like](TODO) AI through my browser.

If I can't play against my friends...

...why not _BUILD MY OWN FRIEND?_