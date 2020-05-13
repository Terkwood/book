# Privileged mode: Running ctop in docker under SELinux

Some programs like `ctop` are nice to run using docker containers, so that you don't have to manually download a binary and copy it into `/usr/local/bin`, and leave it in a sad little corner, unmanaged by `apt` or `yum`.

But if you run an SELinux-enabled linux distribution, you'll find that running `ctop` [as the documentation suggests](https://github.com/bcicen/ctop), fails:

```sh
# THIS DOESN'T WORK :(
docker run --rm -ti \
  --name=ctop \
  --volume /var/run/docker.sock:/var/run/docker.sock:ro \
  quay.io/vektorlab/ctop:latest
```

🐳 🐳 🐳

```text
ctop - error ───────
  │                                                                                 │
  │  [12:54:15 UTC] attempting to reconnect...                                      │
  │                                                                                 │
  │  [12:54:16 UTC] Get http://unix.sock/info: dial unix /var/run/docker.sock: con  │
  │  nect: permission denied                                               
```

What's going on here?  Presumably SELinux is blocking the `ctop` container's access to information necesarry for monitoring the other containers.

## The Fix

Luckily, there's a very easy fix for this!  You can just run the `ctop` container in privileged mode:

```sh
# THIS WORKS! :)
docker run --privileged --rm -ti   --name=ctop   --volume /var/run/docker.sock:/var/run/docker.sock:ro   quay.io/vektorlab/ctop:latest
```

Now you can see all your favorite containers, interact with their log interfaces, dip into their shells, etc:

```text
  ctop - 12:57:42 UTC   8 containers

     NAME        CID         CPU         MEM         NET RX/TX   IO R/W      PIDS

   ◉  bugout_bot… 8f84fe3983…      1%       2M / 944M 19M / 16M   1M / 0B     6
   ◉  bugout_bug… 16f1479be4…      0%       3M / 944M 1M / 1M     6M / 0B     5
   ◉  bugout_gat… fc951914df…      0%       3M / 944M 56M / 35M   256K / 0B   21
```
