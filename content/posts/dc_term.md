+++
title = "Windows Terminal with Double Commander"
date=2023-11-13
description = "Let's use Windows Terminal + WSL2 with Double Commander!"
showFullContent = false
readingTime = false
+++

# Double Commander
Double Commander is an open source file explorer utility that's pretty similar in spirit with Total Commander. It allows for split views, and other handy functions that are otherwise very missed in explorer.exe.

![](/dc.png)

Double Commander allows you to open terminal using F9. However, I had some trouble figuring out how to open my default WSL2 Ubuntu profile on the new Windows Terminal **with the colorscheme and font preferences intact**. This is a quick note on how to achieve that.


# Settings
Go to Menu bar -> Configuration -> Options -> Tools -> Terminal. Edit 'Command for just runnning terminal'.

```
Command: wt
Parameters: -p "Ubuntu" -d . wsl
```

Hit OK.

## Run!
Now, if you press F9, Windows Terminal should start your Ubuntu profile in the current Windows directory inside WSL2!

![](/dc_term.png)

$$
\blacksquare
\blacksquare
\blacksquare
$$
