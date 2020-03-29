# Building the Project

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

It basically worked.  I spent time [making sure the websocket connection between the browser and the BUGOUT gateway server was solid](https://github.com/Terkwood/BUGOUT/issues/48).  I had to [guarantee that Kafka and all its dependent apps started up in an orderly fashion](https://github.com/Terkwood/BUGOUT/blob/dfb4a63be1b052bdb8dc448993d2feecfd91ce74/game-lobby/src/main/kotlin/Application.kt#L531).  And I enjoyed writing [functional-ish Kafka Streams code in Kotlin](https://github.com/Terkwood/BUGOUT/blob/dfb4a63be1b052bdb8dc448993d2feecfd91ce74/game-lobby/src/main/kotlin/Application.kt#L37):  from a cognitive perspective, it felt clean and tidy, even if the topology graphs quickly got out of hand.

![Game Lobby Message Processing: This is Getting Insane](https://github.com/Terkwood/BUGOUT/tree/unstable/game-lobby#topology)