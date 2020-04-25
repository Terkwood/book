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
docker run --rm -v /tmp/example.yaml:/config.yaml:z quay.io/coreos/fcct:release --pretty --strict /config.yaml > example.ign
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
