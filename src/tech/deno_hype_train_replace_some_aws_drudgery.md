# Deno Hype Train: Replace Some AWS Drudgery

As part of my daily development cycle, I create a couple
of virtual machine images in AWS, then forget about them
for a while.  At some point, I boot them up and use them,
and then throw them away.

Once the instances are running, I have no more need for
the AMIs from which they were created, and, crucially,
I don't want to continue spending on the snapshot volumes
to which the AMIs are registered. 

💸 Snapshots cost money! 💸

So, for the last few days I've found myself following a manual workflow to clean up my newly created machine images and snapshots, and that's a big waste of time.

Instead of using the AWS command-line interface to manage the clean-up of these snapshots and machine images, I decided to write a small script using Deno!  After all, the documentation claims that it's a good fit for use cases where you would otherwise employ a small bash or python script.

Follow along and decide for yourself whether it's an improvement.

## Tedious, Motivating Example

So, this is the painfully slow workflow that I used to clean things up by hand.  Sure, it only takes a minute, or maybe two (if I'm feeling slow).  But keep in mind that I've been doing this every day for the last week... 🤢

First, I find the images I control:

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

OK, here's the fun part.  Let's rewrite this drudgery using the shiniest, new tech! 🦕

