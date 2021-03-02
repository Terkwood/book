# Use Keyboard Shortcut to Focus alacritty in Gnome

[alacritty](https://github.com/alacritty/alacritty) is a 🔥 hot, fast terminal emulator 🔥 written in rust. We've been using it for the last 38 microseconds and really enjoy it.

We want to hit a single key and focus on `alacritty` while using GNOME. If `alacritty` isn't already
open, it should start up. This would give us functionality somewhat equivalent to [Guake](http://guake-project.org/).

As a bonus, we'll use [deno](https://deno.land/) to write a quick helper script and 😇 avoid learning `bash`. 😇

Here's what we did to accomplish this task.

## Install wmctrl

Install `wmctrl` so that we can focus on an existing `alacritty` window using a program.

For debian & ubuntu-flavored linux:

```sh
sudo apt install wmctrl
```

For the redhats:

```sh
sudo yum install wmctrl
```

Start `alacritty` up for just a moment and take note of its handle as given by `wmctrl -xl`:

```text
...snip...
0x04800002  1 Alacritty.Alacritty   mybox Alacritty
...snip...
```

We can then use `wmctrl` to focus on a running instance of `alacritty`:

```sh
wmctrl -xa Alacritty.Alacritty
```

## Create script to raise or start-up terminal

Create a `deno` script to either focus on the existing `alacritty` window, or start a new one:

```ts
// saved to /home/nope/bin/raise_alacritty.ts
const p = Deno.run({
  cmd: ["/usr/bin/pgrep", "alacritty"],
});

const { code } = await p.status();

if (code === 0) {
  await Deno.run({ cmd: ["wmctrl", "-xa", "Alacritty.Alacritty"] }).status();
} else {
  await Deno.run({ cmd: ["alacritty"] }).status();
}
```

## Bind keyboard shortcut

Finally, in gnome settings,
add a keyboard shortcut which runs our script.

![keybinding](https://user-images.githubusercontent.com/38859656/81819687-a3904800-94fd-11ea-8f4e-d66c07d600ad.png)

In our case, we assigned the special MENU button on our keyboard to focus on `alacritty`. We use the default GNOME shortcut (`Super H`/`WindowsKey H`) to hide the window when we're done with it.

## Updated for deno 1.0.0-rc3

Please note that as of deno 1.0.0-rc3, the command line invocation needed to make this work has changed. You now need to use the `run` subcommand:

```sh
deno run --allow-run /path/to/raise_alacritty.ts
```

This article was originally published under an older version, which did not require the subcommand.

```sh
deno  --allow-run /path/to/raise_alacritty.ts
```

If you're having trouble getting this to work under the new version of Deno, try adding `run`!
