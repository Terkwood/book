# Lab Notes

Today we're attempting to reinitialize the entire effort.

## State of Affairs: May 21, 2020

- The [outdoor temp microcontroller](https://github.com/Terkwood/prawnalith/tree/unstable/microcontrollers/area_dht11_esp8266) _lives_.
- We completely abandoned [some work to simplify the data model](https://github.com/Terkwood/prawnalith/pull/98).  Revisiting this would be conscientious, as the work is not difficult.

## Trying rust on the ESP8266

🦀 Ah, rust.  

💸 Ah, the $6 ESP8266 microcontroller.

What could be better than spending hundreds of dollars of potentially billable hours making them work together?

In fact there are several potential ways to go about it, but it looks like there's [a new approach](https://github.com/Terkwood/prawnalith/issues/120) recently published online.

Let's give it a try.  The following procedure will download a gigantic XTENSA toolchain which we can hopefully use to push some
simple rust code onto an ESP 8266.  The docker download is a beefy 5GB!  🐄

```text
$ cat /tmp/bootstrap-esp.sh
#!/bin/bash
docker run -ti -v $PWD:/home/project:z quay.io/ctron/rust-esp create-project

$ sh /tmp/bootstrap-esp.sh
Creating Makefile (Makefile)
Creating esp-idf symlink (esp-idf -> /esp-idf)
Creating cargo config (.cargo/config)
Creating main application wrapper (main/esp_app_main.c)
```
