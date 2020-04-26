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

`1756.` Is there anything else I can trim?

`1822.` 176KB.  And I've trimmed out dead code from none other than the illustrious `Goban.js`.

`1831.` The "dead code" in `Goban.js` wasn't so dead. Almost lost the heat map at end-of-game.  [Still finding places to cut](https://github.com/Terkwood/Sabaki/pull/59)! 178KB.

`2124.` [One last pass](https://github.com/Terkwood/Sabaki/pull/60).

# Mon Apr 13, 2020

`1211.` I am very close to having [#67](https://github.com/Terkwood/BUGOUT/issues/67) finished.  I need the browser to work with all of the server components.  I don't believe there are any bugs or gaps left in the server components.

For the browser:

- Do not check whether BUGOUT is online if the `EntryMethod` is `PLAY_BOT`.
- Go slowly and be deliberate about what you add to the browser codebase.
- `GatewayConn` needs a method `playBot`.
- We need to be careful about the modals.  On one hand, it's nice not to overload the existing modals, if possible.  That said, the eventing code which supports the modals is a pain in the butt to work with.

Let's start by adding `EntryMethod`.  We know that's going to be necessary.

`1239.`  Inching into this effort: [we broke up a huge constuctor](https://github.com/Terkwood/Sabaki/pull/61/files) while adding the new `EntryMethod.PLAY_BOT` option.

`1249.` [Refactored the Color Choice dialog](https://github.com/Terkwood/Sabaki/pull/62) for play-vs-humans to have a clearer name.  I don't want to reuse this dialog entirely for the bot-play.  I'll create a new modal for bot-play, but will overload the `choose-color-pref` event which the original dialog emits.

Let's take a break.

`1304.` [Slipped one more](https://github.com/Terkwood/Sabaki/pull/63) change set into the mix before my break.  This is good.  The audits that NPM had been complaining about were taking up more cognitive space than necessary -- just some pesky dev deps. ☕ BREAK TIME! ☕

# Tue Apr 14, 2020

`1204.` We can add some Modals.

`1737.` We added a [wait for bot modal](https://github.com/Terkwood/Sabaki/pull/64).  Not actually using it, yet, but it's minimal and should work fine for our purposes.

# Fri Apr 17, 2020

`1635.` I plan to refactor some of our multiplayer callbacks into a less-annoying, evented thing.   [I also plan to move Sabaki into the monorepo](https://github.com/Terkwood/BUGOUT/issues/219) so that we can tie BUGOUT-wide releases to a given state of Sabaki.


`1642.` It takes seven minutes to write a ticket!?  This is in no way surprising.  It still saves 21 minutes of otherwise-future-time spent thinking about the issue.

`1650.` Chromium just sucks less during the web dev cycle.  I find that I have a hard time getting the freshest version of my changes under FF private mode. 

I HAVE SIX LINES OF CHANGES!

`1708.` [Completed here](https://github.com/Terkwood/Sabaki/pull/65).  Let's consider making some progress on #67.

`1756.` [One more PR to add a modal](https://github.com/Terkwood/Sabaki/pull/66) for color selection for the bot play.  No, it's not yet hooked up to anything!

# Sat Apr 18, 2020

`1221.` We should make a pass at the remaining UI work:

- network calls to `gateway`
- the initial dialog altered
- re-use the existing board-size dialog

We may want to refactor the existing board-size dialog to make it more extensible.  Not sure if it's necessary.  Let's treat its existing behavior as closed.

`1310.` [Extended the board-size dialog init condition](https://github.com/Terkwood/Sabaki/pull/67).

# Sun Apr 19, 2020

`1907.` Success!  First ever bot game.  But something broke in the system and one of the moves computed by KataGo wasn't communicated back to the browser.  Doesn't matter, we'll figure it out.  Happy path worked for the first time!

There is an important `TODO TODO` in gtp.js related to tracking color.

And we can only play black ?  We need to `listenForMove`.

# Mon Apr 20, 2020

`1524.` Whack-a-bug.  Look at `micro-judge`:

```text
[2020-04-20T19:19:06Z ERROR micro_judge::io::stream] MOVE REJECTED: MakeMoveCommand {
        game_id: GameId(
            22ff3e32-4d60-4582-a7d7-607b89e580d3,
        ),
        req_id: ReqId(
            78b33e57-e8ac-425c-93b7-ba0d5b00988b,
        ),
        player: WHITE,
        coord: Some(
            Coord {
                x: 3,
                y: 3,
            },
        ),
    }
```

`1626.` Investigating board.js `coordToVertex` and `vertexToCoord` in Sabaki.

`1629.` TINYBRAIN needs to honor the reversal in the Y-Axis.

`1643.` We won't translate the alphanumeric coordinate

`1730.`  Strip `PASS` out at tinybrain.  Send the `(char, u16)` combo up to `botlink`.  `botlink` can remember the board size and convert back to domain-model `Coord`.

```text
For a 9x9 board,

A1 = Coord { x: 0, y: 8 }
```

`1902.` [PR open to fix all the coords](https://github.com/Terkwood/BUGOUT/pull/223).

# Sun Apr 26, 2020

`0431.`  Trying out [SDKMAN!](https://sdkman.io/) for my gradle install.  Haven't needed gradle on this old workstation, until today.  SDKMAN! is the recommended method for unixy installs of gradle, according to gradle's home page.

It went smoothly...

```sh
sdk install gradle 6.3 
```

```text
Downloading: gradle 6.3

In progress...

############################################################################################# 100.0%

Installing: gradle 6.3
Done installing!


Setting gradle 6.3 as default.
```

That's great.  Debian's default install was a woefully outdated version, which was unusable with the kotlin plugin.

`0455.` [Raised a PR](https://github.com/Terkwood/BUGOUT/pull/243) to emit an event when changelog inits a game state.
