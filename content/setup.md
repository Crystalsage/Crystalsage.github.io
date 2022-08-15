---
title: "Setup"
date: 2021-11-15
draft: false
---
**All technologies are cool**

This page describes the setup that I use in my desktop experience while describing the decisions that were made while selecting the components.  It is irregularly updated.

# OS: Arch Linux
[Arch](https://archlinux.org/) is a fantastic Linux distribution that's pretty lean and mean for all sorts of use cases. It's primary advantage to me is the fact that I have to perform least amount of configuration, in the case where I ever need to reinstall my system. Quite fortunately, it has rarely broken down on me.

# Desktop
## WM: BSPWM
BSPWM is the most vanilla WM experience you can get, in my opinion. I do not care much for the modularity aspect of it. I love i3 as well, but prefer to stick with BSPWM, because it feels more stable.

I have a very terminal oriented workflow and prefer to do everything in terminal. 1 window per workspace works for me.

Some components that I use along with BSPWM:

| Component     | Software                      |
| ---           | ---                           |
| Bar           | polybar                       |
| Terminal      | alacritty + fish shell + tmux |
| Notifications | dunst                         |
| Image viewer  | feh                           |
| Music         | mpd + ncmpcpp                 |
| PDFs          | zathura                       |
| Email         | Neomutt                       |



## Text editor: Neovim
Vim fits my terminal oriented workflow really well. I use NeoVim's built-in LSP.

## Writing
1. **Assignments and other academic material** \
$\LaTeX$ is my personal favorite. I use the VimTeX plugin + Zathura. It produces very clean and beautiful PDFs that are otherwise very missed. 

2. **Notes** \
Casual notes are preferred while learning stuff. I take such notes in Markdown. Simple and universal.

*Obiter dictum*: I love both markdown and Org-mode. I alternate between them from time to time, preferring markdown for smaller files and Org-mode when I want to build a knowledge base. However, I have not fully switched to Emacs and I prefer other tools for other functions.
