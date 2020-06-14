# Juneteenth + 1, 2020

Greetings.  We are attempting a dev-gateway build with a t3.xlarge.

Started at 1014 local time.

The dev build on a t3.medium takes a LONG time:  40 minutes plus !  This is because we build a number of things:

- gateway
- reaper
- micro-judge
- micro-changelog
- micro-game-lobby
- bugle

And we rely on CADDY and REDIS images, also!

## Observations

Just that the build is faster.  This is not a measured or accurate statement, but on the other hand we've done packer builds over and over using t3 medium, because we're stingy!  And the experience with XL is moving along at a better pace.

## Goal 🥅

WE WANT TO BUILD `tinker/streams` BRANCH ON THE LAUNCHED GATEWAY INSTANCE!

We want to move all rust stream based services to implementations that do not require tracking entity IDs on our own.  We will use `XREADGROUP` and let Redis track the IDs.  

We will need to write code which performs creation of consumer groups


