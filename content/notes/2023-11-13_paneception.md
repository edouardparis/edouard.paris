+++ 
title = "Paneception" 
date = "2023-11-13"
slug = "paneception"
+++

```ascii
┌────────────────────────┐
│                    sway│  Paneception
│ ┌───────────────┐      │  “Downward is the only way forward.”
│ │           tmux│      │
│ │ ┌──────┐      │      │
│ │ │   vim│      │      │
│ │ │      │      │      │
│ │ │      │      │      │
│ │ └──────┘      │      │
│ └───────────────┘      │
└────────────────────────┘

```

I reinstalled Tmux.

I had been using i3, then Sway, since 2017 to navigate through window panels
and open a terminal quickly on demand for swift command execution. But after
installing NixOS on my laptop, I wanted to encapsulate libraries, required
services, and environment variables of my personal and work projects into Nix
files that I could load with `nix-shell`. When opening a new Kitty terminal
window with Sway, the new instance does not inherit the state of the previous
shell. I could have a simple script in the Sway configuration that, on the new
window event, moves the user to the working directory and triggers the
`nix-shell` command again if a `default.nix` file is present. However, I don’t
really like to rely on extra configuration of my window manager to manage my
programming workflow.

{{< sidenote >}}
> ¹ I switched my primary development environment to a graphical NixOS VM on a
> macOS host. It has been wonderful. I can maintain the great GUI ecosystem of
> macOS, but all development tools are in a fullscreen VM. One person said, “It
> basically replaced your terminal app,” which is exactly how it feels.
> [@mitchellh](https://twitter.com/mitchellh/status/1346136404682625024)

² [Mitchell Hashimoto’s NixOS configuration](https://github.com/mitchellh/nixos-config)
{{</ sidenote >}}

In fact, I was really inspired by [Mitchell
Hashimoto](https://mitchellh.com/)’s experience with Nix¹². Simply put, a
Nix-configured virtual machine delivers portability and reproducibility that
guarantee a perfectly controlled experience on very good hardware; in his case,
an Apple MacBook Pro. What if one day I wanted to develop with my full Neovim
configuration, complete with all my shell aliases and scripts, on a virtual
machine running on a lightweight iPad? By the time I decide to buy one, the
performance will likely surpass that of my three-year-old laptop. What if I
wanted to code from a distant VPS?

To maintain a simple environment, I don’t want window manager dependencies in
it. I want to try to develop and manage projects from a single workspace pane,
hence my return to the trusty `tmux`. A `tmux` session can open and close
nested panes with the `nix-shell`-inherited sandboxing.

I could also revive my ossified Vim skills to manage everything from the editor
in multiple buffers, but for now, the courage is lacking.

Some current commands:

|                     | vim         | tmux       | sway              |
|---------------------|-------------|------------|-------------------| 
| New vertical pane   | `:vsplit`   | `Ctrl+a %` | `Mod+Enter`       | 
| New horizontal pane | `:split`    | `Ctrl+a "` | `Mod+v Mod+Enter` |
| Close current pane  | `:q`        | `Ctrl+x y` | `Mod+Shift+q`     |
| Move cursor top     | `Ctrl+w k`  | `Ctrl+a k` | `Mod+k`           |
| Move cursor bottom  | `Ctrl+w j`  | `Ctrl+a j` | `Mod+j`           |
| Move cursor left    | `Ctrl+w h`  | `Ctrl+a h` | `Mod+h`           |
| Move cursor right   | `Ctrl+w l`  | `Ctrl+a l` | `Mod+l`           |
