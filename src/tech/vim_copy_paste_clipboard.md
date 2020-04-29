# copy pasting to system clipboard with vim

It's difficult!

We saw some [confirmation that it isn't super easy to use](https://github.com/alacritty/alacritty/issues/2518).

[We read this thread](https://vi.stackexchange.com/questions/84/how-can-i-copy-text-to-the-system-clipboard-from-vim).

We checked for the presence of `+clipboard`:

```text
:echo has('clipboard')
```

...and saw a Big Fat Zero!


## Resolution

We installed `vim-gtk` on our Debian box, using `apt`.

We can now use the "copy on select" feature.  Whenever we select with vim, we can paste out to other applications using the middle mouse button (by default) in Debian/X11.

It's unwieldy, but it works.  We can potentially remap that to a reasonable keystroke combination.


