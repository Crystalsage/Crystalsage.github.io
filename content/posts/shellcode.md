+++
title = "Importing shellcode in IDA and Ghidra"
date=2023-11-13
description = "Disassembling shellcode in IDA and Ghidra by importing it from file."
showFullContent = false
readingTime = false
draft = true
+++

# Why
During reverse engineering, I often run into CTF challenges which output shellcode which requires further analysis. My approach to this involved dynamic analysis with x64dbg and CyberChef's x86 disassembler. 

TODO: Put Cyberchef's screenshot here.

That was, until I started feeling that there must be a better way to disassemble. Turns out you can import and disassemble shellcode in IDA and Ghidra, and even decompile it. I discovered these after Flare-on 10 CTF.

<div style="padding: 0.5rem;border-top:2px solid white;border-bottom:2px solid white;border-left:2px solid white;border-right:2px solid white;" >
<b style="color: #e27885" >Common pitfall</b>
<hr style="background: white">
Both IDA and Ghidra require the files to be in <b>raw binary </b>format. Exporting shellcode as hex will cause the shellcode to be interpreted as a hex string.
</div>

## IDA
1. File->Load file->Additional binary file
2.  Options

Loading segment = 0x0
Loading offset = 0x69000 (arbitrary)

## Ghidra


Happy hacking! :) 
$$
\blacksquare
\blacksquare
\blacksquare
$$
