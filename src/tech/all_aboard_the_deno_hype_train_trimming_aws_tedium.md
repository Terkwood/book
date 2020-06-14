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

Fast-forwarding a bit, we can streamline our code by pulling out the `JSON.parse` call used with every AWS CLI invocation, and declaring it as a function:

```typescript
const parseProcessOutput = async (p: Deno.Process) =>
  JSON.parse(new TextDecoder().decode(await p.output()));
```

We also wanted to stop "typing", "all", "our", "arguments", "to", "the", "Deno.run", "cmd" as comma-separated strings, because we're lazy 👼:

```typescript
const awsEc2Cmd = (argStr: string) => {
  let o = [];
  o.push("/usr/bin/aws");
  o.push("ec2");
  for (let s of argStr.split(" ")) {
    o.push(s);
  }

  return o;
};
```

With these little helpers declared, cleaning up our expen$ive image snapshots is now easy:

```typescript
const dsp = await runOrExit(
  {
    cmd: awsEc2Cmd("describe-snapshots --owner self"),
    stdout: "piped",
  },
);

const { Snapshots } = await parseProcessOutput(dsp);

for (let { SnapshotId } of Snapshots) {
  await runOrExit(
    {
      cmd: awsEc2Cmd(`delete-snapshot --snapshot-id ${SnapshotId}`),
      stdout: undefined,
    },
  );
}
```

That's fully HALF of our daily dev trash that we generate.  We also tend to run a couple of instances in our AWS dev environment.  We want to terminate both of those instances.  One of the two will usually have an elastic IP associated with it, so we make sure to release that IP and avoid charges.

Here's the complete script, which depends on our above helpers being defined in a `procs.ts` file: 

```typescript
import { runOrExit, parseProcessOutput, awsEc2Cmd } from "./procs.ts";
import { config as loadEnv } from "https://deno.land/x/dotenv@v0.3.0/mod.ts";

console.log(loadEnv({ safe: true, export: true }));

// This is the instance tag "Name", used to identify
// our dev environment instances.
// It's loaded from a .env file which looks like this:
//
// KEY_NAME=my-fancy-dev-instance-tag
const KEY_NAME = Deno.env.get("KEY_NAME");

let instsDescd = runOrExit(
  { cmd: awsEc2Cmd("describe-instances"), stdout: "piped" },
);

let addrsDescd = runOrExit(
  { cmd: awsEc2Cmd("describe-addresses"), stdout: "piped" },
);

const { Reservations } = await parseProcessOutput(await instsDescd);

let instancesToTerminate = [];
for (let { Instances } of Reservations) {
  for (let { InstanceId, KeyName } of Instances) {
    if (KEY_NAME === KeyName) {
      instancesToTerminate.push(InstanceId);
    }
  }
}

const { Addresses } = await parseProcessOutput(await addrsDescd);

let addressesToRelease = [];
for (let { InstanceId, AllocationId, AssociationId } of Addresses) {
  if (instancesToTerminate.includes(InstanceId)) {
    addressesToRelease.push({ AllocationId, AssociationId });
  }
}

if (addressesToRelease.length > 0) {
  console.log(`Addresses to release  : ${JSON.stringify(addressesToRelease)}`);

  for (let { AssociationId, AllocationId } of addressesToRelease) {
    await runOrExit({
      cmd: awsEc2Cmd(`disassociate-address --association-id ${AssociationId}`),
    });

    await runOrExit({
      cmd: awsEc2Cmd(`release-address --allocation-id ${AllocationId}`),
    });
  }
}

if (instancesToTerminate.length > 0) {
  console.log(
    `Instances to terminate: ${JSON.stringify(instancesToTerminate)}`
  );

  await runOrExit({
    cmd: awsEc2Cmd(
      `terminate-instances --instance-ids ${instancesToTerminate.join(" ")}`
    ),
  });
}

Deno.exit(0);
```

When I run this script, my credit card sighs in glorious relief. 🤓

## The Triggered Clean-Up Nirvana

Yes, we prefer to watch Stargate SG-1 and Fringe at 9pm.  We do not remember to clean up our precious AWS resources.

We need a _cron job_!

It should wake up this x86_64 laptop from sleep, and use our local AWS credentials to trigger the cleanup scripts.

Why not just run this on a local raspberry pi, or something?  We have at least one of those running 24/7, so we wouldn't need to mess with forcing a wakeup from sleep.

Well, it turns out that Deno **can't run on ARM platforms yet**.  Oh well!

As long as we can wake up our power-hungry laptop reliably, we don't mind a little bit of a hacky solution, here.  It beats waking up our _human body_ to take care of such a tedious task!

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

### Add some crontab love 💟

Well, we left a disk sitting around in AWS for a couple of weeks and it cost us a bit of money.  We dithered and didn't bother to release our article.  NOW WE HAVE GREAT RESOLVE!

We shall run a crontab.

We shall always clean up our expensive dev trash!

[We shall google for how to do this and then tell you about it](https://serverfault.com/a/352837).

Entering the local user and editing the crontab file for correctness before committing it

```sh
sudo crontab -u FRIENDLY_USER -e
```

This was our test run.  We tried a deno script which invoked `wall`, to make sure we could run our more complex deno logic for the dev environment & AMI/snapshot cleanups.

![test drive](https://user-images.githubusercontent.com/38859656/84583409-360a5c80-adc6-11ea-8983-5e4925771d85.png)

### Final Plan 📆

- crontab: (as root!) periodically schedule RTC wake at 23:27 
- crontab (as user, three minutes later): destroy all snapshots and AMIs using deno script
- crontab (as user, also three minutes later): destroy any dev environment instances

That's it.  Our laptop is configured to go back to sleep relatively quickly, so we shouldn't burn too much disgusting coal power (don't hate, we live in Indiana 🤢 🏭) after it kicks on.

#### The RTC Wake-Up Spammer used by ROOT Crontab

![rtc spammer as root](https://user-images.githubusercontent.com/38859656/84583855-ba131300-adcb-11ea-9525-439d1a4ad5a1.png)

#### The cleanup crontab for Normal User

![normal user cleanup](https://user-images.githubusercontent.com/38859656/84583924-ca77bd80-adcc-11ea-91c9-a4f98cbca6ff.png)

We need to make sure the `KEY_NAME` var is exposed in the environment.  First attempt above, failed.

Add another pic...

#### Checking for Correctness

We should make sure that the instances are destroyed, their disks destroyed, elastic IP released completely, snapshots destroyed, AMIs destroyed.

Although our overall approach is a hack, we're willing to accept the disorganization... _as long as everything actually works_!

## References and Attributions

[train image](https://ccsearch.creativecommons.org/photos/b66ad5eb-8395-4eaa-a26f-ba680b23f027) by fsse8info is licensed under CC BY-SA 2.0.

The initial nugget of subprocess management originates in the [Deno manual subprocess example](https://deno.land/manual/examples/subprocess).
