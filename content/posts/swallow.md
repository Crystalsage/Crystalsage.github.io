+++
title = "Swallow"
date = "2022-01-13"
tags = ["linux", "bspwm"]
keywords = ["linux", "bspwm"]
description = "This post showcases a window swallowing solution that works universally."
showFullContent = false
readingTime = false
draft = false
+++

Throughout the discussion, `C-x` means Control + x keychord.
# Introduction
Window swallowing is a mechanism in tiling window managers, by which a GUI application's window can replace the terminal window in-place, when the application is called from the terminal. 


This avoids congestion in tiling WMs, and is a much preferred solution than zooming into the said GUI window (Also called *monocle mode* in some WMs)

For instance, consider a scenario where I want to spawn VLC from my terminal. After I call VLC with `vlc ./video.mp4`, I get a VLC window on my screen. But now, there's a leftover terminal window that does nothing, and you can't close it because it's the parent window of VLC.[^1] 

The following diagram illustrates the difference without and with swallowing, respectively:
![Swallow_VLC](/swallow.png)

Popular WMs such as DWM, BSPWM and i3 have patches available for swallowing a window. However, I prefer a native solution over patches. As a bonus, the solution that I was able to come up with is also universal.

# My solution
My solution, whether if inefficient or dumb, most importantly works and has never failed me. 

I combine a BASH script + a Fish shell function (alias) + a Fish shell keyboard shortcut that swallows a window. Code explained below. Please don't mind the names.

## Bash script: `rbg.sh`
```bash
#!/bin/sh

$@ &
disown
```


This is just doing automatically the stuff that would require manual efforts. The first line takes in the entire `argv`, symbolized with `@` variable, puts it in background with `&` and disowns it. 

The script is meant to be placed in one of the directories in your `$PATH`.

## Fish function: `sdf.fish`

```bash
function sdf
  rbg $argv && exit
end
```

Which calls the `rbg` binary with your command.

This function allows us to wrap over the `rbg.sh` script as well as close the actual terminal window. This only exists because I couldn't find a way to both exit the BASH script and close the terminal window without messing with Xorg tools.


## Fish shortcut: goes in `config.fish`
```bash
bind \cs 'fish_commandline_prepend sdf'
```
Which just prepends `sdf` to your command in the terminal. Note that typing `sdf <command>` is kind of awkward and feels unnatural (I always forget to do it), so typing the command first and prepending is the most natural flow.

# Results
Now, when I type `vlc ./video.mp4` and press `C-s` the resulting command becomes `sdf vlc ./video.mp4`, which successfully swallows the terminal when executed. It's instantaneous with really no lag.



[^1]: You can close the window by putting the process to background, disowning it and then closing the terminal. The problem with this approach is that it is slow and just doesn't feel right.

