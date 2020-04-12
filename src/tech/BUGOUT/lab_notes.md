# Last Mile of KataGo (Apr 11, 2020)

At or around [4947cc6](https://github.com/Terkwood/BUGOUT/commit/4947cc6dc910d88202dad45afc3be94b92f6bece)...

## Notes

`1200.` Issue #67 is almost complete.  KataGo responds if the human picks WHITE.  The response is lost because the UI seems broken (undef dialog).

`1500.` If you play a 19x19 with human as WHITE, KataGo is able to send a move back.  Highlighted that 9x9 has wrong board coords attached at some point in the system (Sabaki receives a MakeMove from BUGOUT with a negative coord).

But wait, the second turn is from the human, but looks duplicated ?

```text
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

`1954.` Came back and merged my PR.  Trying again with Sabaki to see where the AI's `made move` event gets dropped.  Gateway and services seem solid enough now.

`2004.` Trying out deno `file_server -p 8000 .` instead of `python -m SimpleHTTPServer`:

`2020.` I am still tracing through Sabaki trying to find out why `gtp.js` function `listenForMove` doesn't get called.  Done for now.

# Easter 2020

Looks like [Sabaki web branch](https://github.com/SabakiHQ/Sabaki/commit/13e35b45aaee11fdf3c0ed400a645a50a9461657) is deprecated.  Let's consider slimming down our forked `unstable` branch to the bare minimum and trying again with KataGo mods.

`1324.` Trimmed some dead classes out of Sabaki.  Looks like `deno file_server` doesn't like to serve our `/?join=ABC` links!  Back to python...

`1350.` [Closed one set of trimmings](https://github.com/Terkwood/Sabaki/pull/53) and [opened another](https://github.com/Terkwood/Sabaki/pull/54).

`1354.` Here are more things we should delete:

- `fileformats/gib.js` and `ngf.js`
- `App.js` section `if (prevState.fullScreen !== this.state.fullScreen)`

`1405.` [Opened a PR to remove update checking](https://github.com/Terkwood/Sabaki/pull/55).  Closed #54.

`1425.` Closed #55 and [opened a change set trimming file format procedures](https://github.com/Terkwood/Sabaki/pull/56).  Deployed for a little bit more testing.

`1501.` Merged #56. Ventured into `enginesyncer.js` unsuccessfully.  Gave up on that change set and [opened another one trimming App.js and main.js](https://github.com/Terkwood/Sabaki/pull/57).

`1516.` Test deploy. I'm close to <200KB package for users to download, which is nice.  The initial build, at the very start of BUGOUT effort, was about 315KB?

```text
bundle.js  191 KiB       0  [emitted]  main
```

`1536.` Almost messed up the QUIT button.  Ko and suicide popups probably won't work, but that's OK.  Still driving the simplicity score on this app _UP_.  [Yet another PR](https://github.com/Terkwood/Sabaki/pull/58). 182KB size.

`1702.` For a minute I was worried that history provider routines in the browser were broken.  No: as usual, history provider seems to intermittently not respond.  A couple of reboots of the container host and it came back.  Continuing to trim aggressively in #58, will redeploy.

`1706.` 180KB bundle! 🌟
