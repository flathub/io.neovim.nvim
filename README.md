# Neovim Flatpak

Unofficial Flatpak build of [Neovim](https://neovim.io). Neovim runs inside a sandbox, so a few
things work differently from a regular install. This page covers the usual questions.

Found a bug in the packaging? Open an issue here. Bugs in Neovim itself go to
[neovim/neovim](https://github.com/neovim/neovim/issues).

## Where your files go

Flatpak gives the app its own home for config and data. Put your config in the left column, not in
`~/.config/nvim`.

| Inside the sandbox     | On your disk                                  |
| ---------------------- | --------------------------------------------- |
| `~/.config/nvim`       | `~/.var/app/io.neovim.nvim/config/nvim`       |
| `~/.local/share/nvim`  | `~/.var/app/io.neovim.nvim/data/nvim`         |
| `~/.local/state/nvim`  | `~/.var/app/io.neovim.nvim/.local/state/nvim` |
| `~/.cache/nvim`        | `~/.var/app/io.neovim.nvim/cache/nvim`        |

To keep using your existing config, symlink it:

```sh
mkdir -p ~/.var/app/io.neovim.nvim/config
ln -s ~/.config/nvim ~/.var/app/io.neovim.nvim/config/nvim
```

Distributions like LazyVim, NvChad or kickstart.nvim work as usual. Clone them into
`~/.var/app/io.neovim.nvim/config/nvim` instead of `~/.config/nvim`.

## Compilers, language servers and SDK extensions

The sandbox does not see the tools installed on your system. You get the Freedesktop SDK (gcc, make,
python, git...) plus whatever you add as SDK extensions. Without Node.js, for example, Mason can't
install most language servers.

Install the extensions you need. Their branch has to match the runtime this app uses (currently
`25.08`):

```sh
flatpak install flathub org.freedesktop.Sdk.Extension.node24//25.08
flatpak install flathub org.freedesktop.Sdk.Extension.rust-stable//25.08
flatpak install flathub org.freedesktop.Sdk.Extension.golang//25.08
```

Then tell Neovim to load them. Set it once with an override so you don't need to export it every
time:

```sh
flatpak override --user --env=FLATPAK_ENABLE_SDK_EXT=node24,rust-stable,golang io.neovim.nvim
```

Use `*` to load every installed extension. `flatpak search Sdk.Extension` lists what Flathub has.

When the app moves to a new runtime, Neovim opens a notice on the next start. Install the
extensions again for the new branch.

### Packages installed with npm, pip and cargo

These go to `~/.var/app/io.neovim.nvim/data` (`node`, `python`, `cargo`) and the wrapper adds their
`bin` folders to `PATH`. They don't mix with packages on your host. Set
`FLATPAK_ISOLATE_PACKAGES=0` to turn this off.

### A shell with the same environment

The extensions and the `PATH` setup only apply when Neovim starts. `flatpak run --command=sh` gives
you a plain shell without them. Open `:terminal` inside Neovim instead.

## Running commands on the host

Some tools only exist on your system (tmux, podman, a toolbox, a different shell). Allow the app to
talk to Flatpak:

```sh
flatpak override --user --talk-name=org.freedesktop.Flatpak io.neovim.nvim
```

Then run them with `flatpak-spawn --host`:

```sh
flatpak-spawn --host ls
flatpak-spawn --host --env=TERM="$TERM" zsh
```

Without that permission you get `Portal call failed: org.freedesktop.DBus.Error.ServiceUnknown`.

This permission lets Neovim, and any plugin you run, execute anything on your host. Grant it only if
you need it.

## Launching from your shell

An alias like `alias vi='flatpak run io.neovim.nvim'` breaks tab completion in zsh: it completes
`flatpak` arguments instead of file names. Use a function:

```sh
nvim() { flatpak run io.neovim.nvim "$@"; }
```

Or put a script called `nvim` in `~/.local/bin`:

```sh
#!/bin/sh
exec flatpak run io.neovim.nvim "$@"
```

## Clipboard

On Wayland, Neovim uses `wl-copy`, which comes with the app. Some compositors hide the clipboard
protocol from sandboxed apps, and you may see a quick flicker when you yank. If your terminal
supports OSC 52, switch to it:

```lua
vim.g.clipboard = 'osc52'
```

Many terminals block reading the clipboard over OSC 52, so `"+p` may not paste what you copied
elsewhere. Use your terminal's paste shortcut (usually `Ctrl+Shift+V`) instead.

## Extra tools as extensions

The app has an extension point at `io.neovim.nvim.tool`. Extensions there install into
`/app/tools/<name>` and the wrapper adds their `bin` to `PATH`. If you package a tool this way, open
an issue so we can list it here.
