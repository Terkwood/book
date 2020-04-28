# Experimenting with Fedora Core OS

As part of [BUGOUT](https://github.com/Terkwood/BUGOUT/issues/176), we want to attempt supporting Fedora CoreOS.  We previously ran our stateful container deployment on CoreOS Container Linux and enjoyed the automatic OS upgrades. It used a relatively small amount of RAM. We were too lazy to move beyond using `docker-compose`, and that wasn't a problem:  after writing a couple of tiny `systemd` configs, our system always started up reliably.

But CoreOS Container Linux is being put to pasture, so we've decided to try the new thing. 

## First Steps with FCCT

We need to create an ignition file which is intended to be written once, and valid for the life of the image.

See [some docs](https://github.com/coreos/fcct/blob/master/docs/getting-started.md) for getting started with this configuration tool.  

They recommend using `podman` instead of `docker`, but there's no `snap` install available that doesn't warn about potentially stomping on our localdev ❤️ Debian ❤️ system.

```sh
sudo snap install --edge podman
```

```text
error: The publisher of snap "podman" has indicated that they do not consider this revision to be
       of production quality and that it is only meant for development or testing at this point. As
       a consequence this snap will not refresh automatically and may perform arbitrary system
       changes outside of the security sandbox snaps are generally confined to, which may put your
       system at risk.

       If you understand and want to proceed repeat the command including --devmode; if instead you
       want to install the snap forcing it into strict confinement repeat the command including
       --jailmode.
```

Use `docker` to produce the ignition file, instead.  First we created an example YAML file that fcct could read.  This was a bit unclear based on the current state of the documentation, as we used `.yaml` for the file extension instead of `.fcc`.

```yaml
# local example.yaml
variant: fcos
version: 1.0.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc...
```

```sh
docker pull quay.io/coreos/fcct:release
docker run  -i --rm quay.io/coreos/fcct:release --pretty --strict <  input.yaml > example.ign
```

As promised, this command output an ignition file:

```json
{
  "ignition": {
    "config": {
      "replace": {
        "source": null,
        "verification": {}
      }
    },
    "security": {
      "tls": {}
    },
    "timeouts": {},
    "version": "3.0.0"
  },
  "passwd": {
    "users": [
      {
        "name": "core",
        "sshAuthorizedKeys": [
          "ssh-rsa AAAAB3NzaC1yc..."
        ]
      }
    ]
  },
  "storage": {},
  "systemd": {}
}
```

## Launching an Instance on AWS

We can launch an instance on AWS.  [See the operating system getting started page](https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/).

The docs currently ask you to use the `aws` command line interface.  I'm too lazy to do that.  I don't want to (re)learn the options for the `ec2 run instances` command.  I just want to plug some values in via the web interface.

To do that, click through the [downloads page](https://getfedora.org/coreos/download?tab=cloud_launchable&stream=stable), look for for your AWS region, click through to the launch instance page within AWS, then look for the "user data" details.

![user data for your igntion config](https://user-images.githubusercontent.com/38859656/80287955-918f6680-8702-11ea-8021-87040838e890.png)

You can enter your ignition config here.

Finally, you launch the instance and connect:

```text
Fedora CoreOS 31.20200407.3.0
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/c/server/coreos/
```

## Setting up the system

Right now we're manually exploring the system to see 
how things feel.

```sh
git clone https://github.com/Terkwood/BUGOUT.git
cd BUGOUT
sudo usermod -aG docker $USER  # we'll fix this in next section
sudo reboot   # reboot helped
```

In `build-giant-dc.sh` we were using `docker-compose`.  Let's try an alternate docker socket.

Grab docke-compose by the [cheater method](https://github.com/circleci/circleci-docs/issues/1323#issuecomment-323620271) for now.

```sh
pushd ~
curl -L https://github.com/docker/compose/releases/download/1.11.2/docker-compose-`uname -s`-`uname -m` > ~/docker-compose
chmod +x ~/docker-compose
sudo mkdir /opt/bin
sudo mv ~/docker-compose /opt/bin/docker-compose
export PATH="$PATH:/opt/bin"
popd
docker-compose -f dc-giant.yml build judge
```

You can use `ctop`:

```sh
docker run -ti -v /var/run/docker.sock:/var/run/docker.sock quay.io/vektorlab/ctop:latest
```

You *cannot* use `toolbox`, because it fails.

You can use `rpm-ostree` to install `htop`:

```sh
sudo rpm-ostree install htop
sudo systemctl reboot  # you need to reboot to get access to it
```

## Exploring Packer

We are clearly going to need to manage the creation of this image.
Let's try using Packer.

Here is their basic guide to [build an image](https://www.packer.io/intro/getting-started/build-image.html).

We can move on to running something a bit more interesting.

You first need to write a program.

```sh
cat >hello.ts
console.log("Welocem Friend 🦕");
```

You need some env vars specified.


```sh
# set_deno_env.sh
export VPC_ID="vpc-deadbeef"
export SUBNET_ID="subnet-bad1dea5"
```

...then...

```sh
source set_deno_env.sh
```

Write some `packer-example.json`

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
        "region":         "us-east-1",
        "vpc_id":         "{{env `VPC_ID`}}",
        "subnet_id":      "{{env `SUBNET_ID`}}"
    },
    "builders": [
        {
            "access_key": "{{user `aws_access_key`}}",
            "ami_name": "packer-linux-aws-demo-{{timestamp}}",
            "instance_type": "t3.micro",
            "region": "{{user `region`}}",
            "vpc_id": "{{user `vpc_id`}}",
	          "subnet_id": "{{user `subnet_id`}}",
            "secret_key": "{{user `aws_secret_key`}}",
            "source_ami_filter": {
              "filters": {
              "virtualization-type": "hvm",
              "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
              "root-device-type": "ebs"
              },
              "owners": ["099720109477"],
              "most_recent": true
            },
            "ssh_username": "ubuntu",
            "type": "amazon-ebs"
        }
    ],
    "provisioners": [
        {
            "type": "file",
            "source": "./welcome.txt",
            "destination": "/home/ubuntu/"
        },
        {
            "type": "file",
            "source": "./hello.ts",
            "destination": "/home/ubuntu/"
        },
        {
            "type": "shell",
            "inline":[
                "ls -al /home/ubuntu",
                "cat /home/ubuntu/welcome.txt"
            ]
        },
        {
            "type": "shell",
            "inline": [
              "sudo apt install -y unzip",
              "curl -fsSL https://deno.land/x/install/install.sh | sh"
              
            ]
        },
        {
            "type": "shell",
            "inline":[
              "export DENO_INSTALL=\"/home/ubuntu/.deno\"",
              "export PATH=\"$DENO_INSTALL/bin:$PATH\"",
              "deno hello.ts"
            ]
        }
    ]
}
```

You'll see a bunch of glorious progress, and finally, an artifact:

```text
==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
us-east-1: ami-eeeeeeeeeeeeeeeea
```

Don't forget to deregister the AMI after you're done!

## Putting It All Together With FCOS

That's all well and good, and we're happy about `packer`.

But we need to usable FCOS image which has a few modifications that `packer`, rather than `ignition` is well-suited to handle.

Create an ignition file which you'll include in the packer config:

```yaml
variant: fcos
version: 1.0.0
passwd:
  users:
    - name: core
      groups: [ docker ]
      ssh_authorized_keys:
        - ssh-rsa AAAA...
storage:
  files:
    - path: /opt/bin/docker-compose
      overwrite: true
      mode: 0755
      contents:
        source: https://github.com/docker/compose/releases/download/1.13.0/docker-compose-Linux-x86_64
        verification:
          hash: sha512-9d2c4317784999064ba1b71dbcb6830dba38174b63b1b0fa922a94a7ef3479f675f0569b49e0c3361011e222df41be4f1168590f7ea871bcb0f2109f5848b897
```

```sh
docker run  -i --rm quay.io/coreos/fcct:release --pretty --strict <  packed.yaml > packed.ign
```

Then write your packer config:

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
        "region":         "us-east-1",
        "vpc_id":         "{{env `VPC_ID`}}",
        "subnet_id":      "{{env `SUBNET_ID`}}"
    },
    "builders": [
        {
            "access_key": "{{user `aws_access_key`}}",
            "ami_name": "fcos-linux-aws-demo-{{timestamp}}",
            "instance_type": "t3.medium",
            "region": "{{user `region`}}",
            "vpc_id": "{{user `vpc_id`}}",
            "subnet_id": "{{user `subnet_id`}}",
            "secret_key": "{{user `aws_secret_key`}}",
            "user_data_file": "packed.ign",
            "source_ami_filter": {
              "filters": {
                "virtualization-type": "hvm",
                "name": "fedora-coreos-31.20200407.3.0",
                "root-device-type": "ebs"
              },
              "owners": ["125523088429"],
              "most_recent": true
            },
            "ssh_username": "core",
            "type": "amazon-ebs"
        }
    ],
    "provisioners": [
        {
            "type": "shell",
            "inline": [
                "sudo rpm-ostree install htop"
            ]
        },
        {
            "type": "shell",
            "inline":[
                "git clone https://github.com/Terkwood/BUGOUT.git"
            ]
        }
    ]
}
```

...and we have a baked image!

We used docker-compose 1.13 because python3.7 fails to link to libcrypt.so.1 on Fedora 31.  :\

## Launch Templates Are Helpful

So [we created one](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#launch-instance-from-launch-template).

We can then launch the instance like so:

```sh
aws ec2 run-instances --launch-template LaunchTemplateId=lt-0000,Version=2 --image-id ami-0000 --subnet-id subnet-deadbeef
```

You can override user data at the command line.

## Completing the ignition

Next up, write some `systemd` data and figure out how
to seat some `dev.env` files as `.env` files.
