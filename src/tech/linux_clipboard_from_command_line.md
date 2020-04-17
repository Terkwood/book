# Copying to the clipboard from the Linux command line

Just use `xsel`.

Debian install:

```sh
sudo apt install xsel
```

Demo:

```sh
cat /tmp/takeout-order | xsel --clipboard
```
