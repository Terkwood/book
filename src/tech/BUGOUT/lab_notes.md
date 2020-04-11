# Last Mile of KataGo (Apr 11, 2020)

At or around [4947cc6](https://github.com/Terkwood/BUGOUT/commit/4947cc6dc910d88202dad45afc3be94b92f6bece)...

## Notes

`1200.` Issue #67 is almost complete.  KataGo responds if the human picks WHITE.  The response is lost because the UI seems broken (undef dialog).

`1500.` If you play a 19x19 with human as WHITE, KataGo is able to send a move back.  Highlighted that 9x9 has wrong board coords attached at some point in the system (Sabaki receives a MakeMove from BUGOUT with a negative coord).

But wait, the second turn is from the human, but looks duplicated ?

```
[2020-04-11T19:02:56Z INFO  micro_judge::io::stream] ACCEPTED: MoveMade { game_id: GameId(abb65823-febe-49fd-9d61-9b9374cf943f), reply_to: ReqId(a848cad4-6d46-44b9-8389-289427307f3d), event_id: EventId(c13797b3-5250-41f8-ac57-0005a59b1d6f), player: BLACK, coord: Some(Coord { x: 2, y: 3 }), captured: [] }


[2020-04-11T19:03:04Z INFO  micro_judge::io::stream] ACCEPTED: MoveMade { game_id: GameId(abb65823-febe-49fd-9d61-9b9374cf943f), reply_to: ReqId(3c34a9e5-911c-4d6f-9030-abd43aca4f75), event_id: EventId(2415ac47-2ece-423c-a9e9-acb861d11fb5), player: BLACK, coord: Some(Coord { x: 2, y: 3 }), captured: [] }
```

`1522.` Hacked Sabaki (`3fc20268`) to avoid sending duplicates.  Now I see that game state changelog isn't being propagated correctly.  Time for a break.

`1542.` Time to consolidate my progress in `gateway` and make a commit.

`1636.` I don't understand why `micro-changelog` tracks game states correctly, but `micro-judge` doesn't.  The changelog service XADDs faithfully.  `micro-judge` seems to miss the update.

`1705.` Judge is failing to update ITS game state changelog entry.  Changelog service is reading its own stream and updating its repo correctly.  BAD Judge.  GOOD Changelog!

`1716.` Judge appears to be spinning in its stream processor. :\

`1738.` I can see that the XRead EntryID in judge isn't updated for game state.

`1755.` Is the Redis XREAD deserialization type correct? 

```rs
pub type XReadResult = Vec<HashMap<String, Vec<HashMap<String, HashMap<String, String>>>>>;
```

`1812.` Commit `e73e8e2` has Judge assuming an XREAD with `Vec<u8>` data, not `String`.  Gonna be annoying to pull this out if it doesn't help, which it probably won't.

`1818.`  It worked!  Judge now updates its game state changelog correctly.  All it cost was an extra round of `.clone()`s.  Well, I still can't progress past move two in Sabaki, but that's probably unrelated.  I'm going to commit `micro-judge` and `micro-changelog` now, since they've moved forward.

`1828.` Raised my PR and am shutting down this very pricey `t3.medium` that I use for development. [Isnt-it-romantic](https://github.com/Terkwood/BUGOUT/pull/216).
