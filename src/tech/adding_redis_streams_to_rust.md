# Adding Redis Streams to Rust 🏞️ 🦀 

I recently [worked on a pull request](https://github.com/mitsuhiko/redis-rs/pull/319) which adds [Redis Streams](https://redis.io/topics/streams-intro) capabilities to [redis-rs](https://github.com/mitsuhiko/redis-rs), the most popular Redis client lib in the rust community.  The overwhelming majority of the effort was contributed by the community, not by me:  I simply drafted the pull request which combines the two existing works.  This required some very light touch-up, and adding a few examples of how the new API works.

The PR is just now entering its final review phase, as several of us have wanted to put our best foot forward before pestering the hard-working maintainers of redis-rs.  

Still, I'm feeling the hype. 🔥  What is Redis Streams? Why do we need it in Rust?  Does it have anything to do with Kafka Streams?  And can we share any real-life examples?

Please read on!

## What is Redis Streams?

To give a little bit of background, Redis Streams was released as part of Redis 5.0, all the way back [in Oct 2018](https://redislabs.com/blog/redis-5-0-is-here/).

[antirez posts a great explanation of the new features](http://antirez.com/news/128).

You can think of Redis Streams data as a way to communicate _time-indexed_ data among processes.  Oversimplifying this, you can imagine that each stream is a CSV with a timestamp, flowing through memory.  Digging deeper, Redis Streams exposes functionality that's more advanced than basic pub/sub, as you can have multiple consumer groups eat up a single stream:  even if one consumer happens to be faster than the others, Redis will rationally distribute the data.  

Too, unlike Redis's pub/sub mechanism, Redis Streams are somewhat durable.  If you miss a publication for some reason, it's still available in the stream, with limits; a stream can have a cap on the amount of messages allowed to accumulate.

## Why Does Rust Need Redis Streams?

Regardless of how quickly we all adopt the Redis Streams, the new features are over a year old, so there was [some support expressed](https://github.com/mitsuhiko/redis-rs/issues/162#issuecomment-627459529) for including the streams commands within the main library.  We've implemented it as a Feature, so that if you want to reduce your compile time 🦀, you can explicitly disable streams support.


## Off-the-Cuff Comparison with Kafka Streams

After looking into Redis Streams on my own, I did some initial comparisions with Kafka Streams and came to some conclusions:

- Redis Streams is a subset of the Redis server API.  It is a set of commands exposed by the server, not a separate lib.  OTOH, Kafka Streams is a combination of the server (Kafka) combined with a JVM-based framework/lib (Kafka Streams).  
- Kafka Streams apps coordinate data partitioning on the client side, while in Redis Streams, the server decides which consumer group gets which slice of data.
- Naive Kafka Streams apps consume a LOT of RAM, so if you're stingy about that sort of thing (and don't plan on scaling to infinity), Redis Streams can be a great fit
- The data types exposed by Redis Streams are somewhat deeply nested, and can take a little while to get used to
- Tracking your position in the stream can get a bit unwieldy compared to Kafka Streams apps
- Redis Streams is quite low-level compared to Kafka Streams and its pleasant, declarative programming style

So, despite the similar naming conventions, the use cases for Kafka Streams vs Redis Streams don't overlap very well.  Redis works great when you have one or two boxes, each with an enormous amount of RAM.  Kafka is built for massive, horizontal scale.

## Practical Comparison

I had already written several Kafka Streams applications to process game states and various player-coordination functions for BUGOUT, our implementation of the ancient board game Go/Baduk. I went ahead and rewrote some of this functionality using rust and Redis Streams.  You can see some direct comparisons of how one might structure a Redis Streams app versuse a Kafka Streams app.

- judging moves in [Rust/Redis Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/micro-judge) vs [Kafka Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/judge)
- assigning players to game instances in [Rust/Redis Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/micro-game-lobby) vs [Kafka Streams](https://github.com/Terkwood/BUGOUT/tree/unstable/game-lobby)

In the Redis Streams implementations, we wrote "entry ID repositories" which allowed us to track the time index of the most-recently read entry from the stream.  This is where the declarative style of Kafka Streams apps really stood out as an advantage:  we could just focus on the logic required by our game system, and didn't have to write quite as much boilerplate.

But our micro Redis Streams written in rust were _miniscule_ in terms of their memory consumption: in a cold system, just after startup, the micro-judge and micro-game-lobby apps take up about 1MB of RAM, while the Kafka Streams apps ⚠️ usually initialize at 100MB+ ⚠️.

Note that these Redis examples don't actually use the nicer API that's discussed as the main focus of this article -- generally we're using the lowest-level interface which specifies Redis commands using strings.  We'll work on upgrading these files after the aforementioned PR is merged! 

## Conclusion

From my own, narrow POV, Redis Streams was quite nice to use, especially considering that I'm the one paying for my app's AWS hosting, and want to conserve memory!

If you're exicited about seeing the Redis Streams support finally make it into the Rust ecosystem in a nice way, please [dust the PR with emojis](https://github.com/mitsuhiko/redis-rs/pull/319), or better yet, with critical review. 🔬

If you're using Redis Streams in your own work, we'd love to hear from you.

We're also very much open to fact-checking on our claims in this article, and interested in hearing criticisms of Redis Streams, if you have them!

