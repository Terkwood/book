![ALL ABOARD](https://user-images.githubusercontent.com/38859656/82162993-328ebe80-9876-11ea-930f-5f5d881f75a1.jpg)

# All Aboard the Deno Hype Train: Trimming AWS Tedium

This article explores the use of [Deno](https://deno.land/) to automate repetitive AWS clean-up tasks.  We conclude by installing a `cron` job that wakes up our laptop and disposes of our AWS testing resources every night.


Please be advised that this article is for educational purposes only.  

If you blindly follow all the instructions, you'll end up deleting all your AMIs and snapshots.  ⚠️


## Detail

As part of our daily development cycle, we create a couple
of virtual machine images in AWS, then forget about them
for a while.  At some point, we boot them up and use them,
and then throw them away.

Once the instances are running, we have no more need for
the AMIs from which they were created, and, crucially,
we don't want to continue spending on the snapshot volumes
to which the AMIs are registered. 

💸 Snapshots cost money! 💸

So, for the last few days we've followed a manual workflow to clean up our newly created machine images and snapshots, and that's a big waste of time.

Instead of using the AWS command-line interface to manage the clean-up of these snapshots and machine images, we decided to write a small script using Deno!  After all, the documentation claims that it's a good fit for use cases where you would otherwise employ a small bash or python script.

Follow along and decide for yourself whether it's an improvement.

## Tedious, Motivating Example

So, this is the painfully slow workflow that we used to clean things up by hand.  Sure, it only takes a minute, or maybe two (if we're feeling slow).  But keep in mind that we've been doing this every day for the last week... 🤢

First, we find the images we control:

```sh
aws ec2 describe-images --owners self|grep ami
```
▶️ ▶️ ▶️

```text
"ImageId": "ami-0aaaaaaaaaaaaaaaa",
"ImageId": "ami-0bbbbbbbbbbbbbbbb",
```

Deregister (disassociate) their respective volumes:

```sh
aws ec2 deregister-image --image-id ami-0aaaaaaaaaaaaaaaa
aws ec2 deregister-image --image-id ami-0bbbbbbbbbbbbbbbb
```

Now that the images are deregistered, find all of our snapshots:

```sh
aws ec2 describe-snapshots --owner self | grep snap-
```

▶️ ▶️ ▶️

```text
"SnapshotId": "snap-0cccccccccccccccc",
"SnapshotId": "snap-0dddddddddddddddd",
```

Finally, destroy these snapshots 

```sh
aws ec2 delete-snapshot --snapshot-id snap-0cccccccccccccccc
aws ec2 delete-snapshot --snapshot-id snap-0dddddddddddddddd
```

What colossal a waste of time!

## Pleasantly Hyped Automation 

OK, here's the fun part.  Let's rewrite this junk using the shiniest,  newest tech! 🦕 

Our workflow requires that we look up both Amazon Machine Image (AMI) IDs, as well as Snapshot IDs.  We deregister the AMIs one by one, and we delete the snapshots one by one, as well.

Executing a command via subprocess is easily accomplished using `Deno.run`.  Here we use `stdout: "piped"` because we'll want to capture the output from the command and manipulate it:

```typescript
const p = Deno.run({ cmd: ["/usr/bin/aws", "ec2", "describe-images", "--owners", "self"], stdout: "piped" });

const { code } = await p.status();

if (code !== 0) {
    const rawError = await p.stderrOutput();
    const errorString = new TextDecoder().decode(rawError);
    console.log(errorString);
    Deno.exit(code);
}
```

If we were to run this command directly in a terminal, we'd see
something like this:

```json
{
    "Images": [
        {
            "Architecture": "x86_64",
            "CreationDate": "1970-01-01T08:25:24.000Z",
            "ImageId": "ami-0aaaaaaaaaaaaaaaa",
            // ... SNIP ...
        },
        {
            "Architecture": "x86_64",
            "CreationDate": "1970-01-01T08:15:13.000Z",
            "ImageId": "ami-0bbbbbbbbbbbbbbbb",
            /// ... SNIP ...
        }
    ]
}
```

Using `grep` to pull out the `ImageId` field will leave us with an ugly line of text which we then need to further trim down to the actual ID. And we're too lazy to write a proper regex. 🌝

Thankfully, Typescript makes this dead simple for us, since it handles JSON very naturally:


```typescript
const { Images } = JSON.parse(new TextDecoder().decode(await p.output()));

for (let { ImageId } of Images) {
  await Deno.run({ 
        cmd: ["aws", "ec2", "deregister-image", "--image-id", ImageId] 
    });
}
```

## The Triggered Clean-Up Nirvana

Yes, we prefer to watch Stargate SG-1 and Fringe at 9pm.  We do not remember to clean up our precious AWS resources.

We need the _cron job_!

It should wake up this x86_64 laptop from sleep, and use our local AWS credentials to trigger the cleanup scripts.

Why not just run this on a local raspberry pi, or something?  We have at least one of those running 24/7, so we wouldn't need to mess with forcing a wakeup from sleep.

Well, it turns out that Deno **can't run on ARM platforms yet**.  Oh well!

What about a cloud deployment?

ARE YOU KIDDING ME?

💸 We 💸 Can't 💸 Give 💸 Jeff 💸 All 💸 The 💸 Money! 💸

### Try Out RtcWake

We'll attempt to [use RtcWake to jolt our laptop into consciousness](https://www.addictivetips.com/ubuntu-linux-tips/automatically-wake-linux-up-from-sleep/).


🎵 🔈 First, put the music on nice and loud... 🎹 👾

```sh
# install debian & ubuntu
sudo apt install -y util-linux

# put the laptop to sleep for 10 seconds, then resume
sudo rtcwake -u -s 10 -m mem
```

Blackout... 🙈

...and in 10 seconds, we heard our tunes pop right back into place! 👂 🏁

### Scheduled Wake-Up

We're gonna need to do this by cron, so we need to be able to

```sh
sudo rtcwake -m no -l -t $(date +%s -d 'today 19:15')
```

⌚️ Try this at 19:14, then wait...

🌞 No problems!

## References and Attributions

[train image](https://ccsearch.creativecommons.org/photos/b66ad5eb-8395-4eaa-a26f-ba680b23f027) by fsse8info is licensed under CC BY-SA 2.0.

The initial nugget of subprocess management originates in the [Deno manual subprocess example](https://deno.land/manual/examples/subprocess).


