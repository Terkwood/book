# Fast scrolling with tmux and alacritty

![be fast](https://user-images.githubusercontent.com/38859656/80623620-78472c80-8a18-11ea-933d-00b52178f809.png)

We want to have a nice experience when scrolling the terminal buffer using both alacritty and tmux on Debian linux.

When we run alacritty and tmux together and there's a lot of text on the screen (say, by invoking `tree`), we need to hit the following tmux key sequence to scroll back through the terminal history: 

```text
Ctrl+B 
[
PageUp
```

This is difficult.  Others have [asked about this](https://github.com/alacritty/alacritty/issues/1194).

We followed the workaround advice given by the author.  We disabled faux scrolling in `.tmux.conf`: 

```text
set -ga terminal-overrides ',*256color*:smcup@:rmcup@'
```

In `.alacritty.conf`, the default scrolling key combination, `Shift+PageUp`, isn't too bad.  We can just leave it commented to use it, but if we want to try our own settings, we can:

```text
key_bindings:
#
# ...snip...
#
- { key: PageUp,   mods: Shift, action: ScrollPageUp,   mode: ~Alt       }
- { key: PageDown, mods: Shift, action: ScrollPageDown, mode: ~Alt       }
- { key: Home,     mods: Shift, action: ScrollToTop,    mode: ~Alt       }
- { key: End,      mods: Shift, action: ScrollToBottom, mode: ~Alt       }
```

## Resolution

With the changes to `.tmux.conf`, we can now start an alacritty terminal, enter tmux, and scroll to our hearts' content, without using a difficult key combination.

🔥 It's FAST! 🔥

We found that scrolling through history with some basic command line apps wasn't negatively impacted. 

When we entered tmux + alacritty, and then dumped a lot of text into `less`, or navigated `man bash`, we scrolled happily and without interruption.

These instructions probably apply to other terminal emulators, as well.

This is a happy time. 😄