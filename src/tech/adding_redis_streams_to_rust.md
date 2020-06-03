![Get Into Teh Streamss](https://user-images.githubusercontent.com/38859656/83622799-7532e500-a55e-11ea-844e-763782e45657.jpg)

# Adding Redis Streams to Rust 💾 🦀 

I recently [worked on a pull request](https://github.com/mitsuhiko/redis-rs/pull/319) which adds [Redis Streams](https://redis.io/topics/streams-intro) capabilities to [redis-rs](https://github.com/mitsuhiko/redis-rs), the most popular Redis client in Rust's ecosystem.  The overwhelming majority of the effort was [contributed by the community](https://github.com/grippy/redis-streams-rs), not by me: I drafted the pull request which combines the two existing works.  In addition to addressing review comments, I added a few examples of how the new API works.

I'm feeling the hype. 🔥  What is Redis Streams? Why do we need it in Rust?  Does it have anything to do with Kafka Streams?  And can we share any real-life examples?

Please read on!

## What is Redis Streams? 🤔

To give a little bit of background, Redis Streams was released as part of Redis 5.0, all the way back [in Oct 2018](https://redislabs.com/blog/redis-5-0-is-here/).

[antirez posts a great explanation of the new features](http://antirez.com/news/128).

You can think of Redis Streams data as a way to communicate _time-indexed_ data among processes.  Oversimplifying this, you can imagine that each stream is a CSV with a timestamp.  Digging deeper, Redis Streams exposes functionality that's more advanced than basic pub/sub, as you can have multiple consumers process a single stream:  even if one consumer happens to be faster than the others, Redis will rationally distribute the data.  

Unlike Redis's pub/sub mechanism, Redis Streams are somewhat durable.  If you miss a message, it's still available in the stream, with limits; a stream will generally have a cap on the amount of messages allowed to accumulate.

## Why Does Rust Need Redis Streams? 🔎

The Streams commands are over a year old, so there was [some support expressed](https://github.com/mitsuhiko/redis-rs/issues/162#issuecomment-627459529) for including them in redis-rs.  

It's great that the community came together and created a [separate lib](https://github.com/grippy/redis-streams-rs) exposing the Redis Streams API.  But Redis Streams is a first-class citizen of the larger Redis API, so it makes sense to include it as part of the leading Redis crate.

We've exposed the new Redis Streams commands as an opt-out feature in redis-rs. If you want to reduce your compile time 🦀, you can explicitly disable streams support. This is the same as how geospatial operators work in redis-rs, so it should be a familiar concept for developers who have experience with the lib and who want to try out Streams.


## Off-the-Cuff Comparison with Kafka Streams 🌽

After looking into Redis Streams on my own, I did some initial comparisions with Kafka Streams and came to some conclusions:

- Redis Streams is a subset of the Redis server API.  The stream commands are exposed by the basic Redis server, not through separate lib.  On the other hand, Kafka Streams is a combination of the server (Kafka) combined with a JVM-based framework/lib (Kafka Streams).
- Kafka Streams apps coordinate data partitioning on the client side, while in Redis Streams, the server decides which consumer group gets which slice of data.  This helps Kafka achieve extraordinary scale: clients can work together to distribute load without the server becoming a bottleneck.
- Naive Kafka Streams apps consume a LOT of RAM, so if you're stingy about that sort of thing (and don't plan on scaling to infinity), Redis Streams can be a great fit.
- The data types exposed by Redis Streams are somewhat deeply nested, and can take a little while to get used to.

Despite the similar naming conventions, the use cases for Kafka Streams and Redis Streams don't overlap as much as you might think.  Redis works great when you have one or two boxes (potentially with an enormous amount of RAM).  Kafka is built for massive, horizontal scale.

## Practical Comparison 🔧

Out of sheer, nerdtastic enthusiasm, I had already written several Kafka Streams applications to process game states and various player-coordination functions for BUGOUT, our implementation of the ancient board game Go (Baduk). I went ahead and rewrote some of this functionality using rust and Redis Streams.  You can see some direct comparisons of how one might structure a Redis Streams app versuse a Kafka Streams app.

- judging moves in [Rust/Redis Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/micro-judge) vs [Kafka Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/judge)
- assigning players to game instances in [Rust/Redis Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/micro-game-lobby) vs [Kafka Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/game-lobby)

The declarative style of Kafka Streams apps really stood out as an advantage:  we could just focus on the logic required by our game system, and didn't have to write quite as much boilerplate.

But our Redis Streams apps written in rust were _miniscule_ in terms of their memory consumption: in a cold system, just after startup, the micro-judge and micro-game-lobby apps take up about 1MB of RAM, while the Kafka Streams apps ⚠️ usually initialize at 100MB+ ⚠️.  Meanwhile, the Redis server itself continues to cruise along with a similarly small main-memory footprint (~3MB in our low-traffic system).

Note that these Redis examples don't actually use the nicer API that's discussed as the main focus of this article -- generally we're using the lowest-level interface which specifies Redis commands using strings.  We'll work on upgrading these files once our [pull request](https://github.com/mitsuhiko/redis-rs/pull/319) is merged! 

## Look Ma, I'm Learning! 🧠

As a result of working the merge, I improved my understanding of the concepts underlying Redis Streams.

Creating an example of `XREADGROUP` command patterns gave me an immediate insight into a [shortcoming of my board game project](https://github.com/Terkwood/BUGOUT/issues/310):  I was maintaining boilerplate code, tracking the time IDs processed in a given stream. This code can be destroyed once I switch from naive `XREAD` to Redis-controlled `XREADGROUP`.

Using the ">" operator in an `XREADGROUP` command tells Redis, "hey, give me only the newest records... and YOU keep track of where I am in the stream!"   This functionality, combined with the automatic `XACK` provided in in the new additions to redis-rs, makes for a nice combination.

## Conclusion 💛

If you're exicited about seeing the Redis Streams support finally make it into the Rust ecosystem in a nice way, please [dust the PR with emojis](https://github.com/mitsuhiko/redis-rs/pull/319), or better yet, with critical review. 🔬

If you're using Redis Streams in your own work, we'd love to hear from you.

### Attribution

Thank you to [Audrey](https://www.flickr.com/photos/audreyjm529/) for the [image used in the header](https://www.flickr.com/photos/98799884@N00/235458062).  It is licensed under [CC by 2.0](https://creativecommons.org/licenses/by/2.0/). I cropped the image in order to make it fit a bit better as a header.
