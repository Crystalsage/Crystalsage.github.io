+++
title = "CSI-CTF Writeup (Part-3)"
date=2022-02-23
description = "The final part of the 3-part writeup for CSI-CTF 2022"
showFullContent = false
readingTime = false
+++

This is the 3rd and final part of the 3-part series for the CSI-CTF writeup. 
Click here to read [1st part](https://crystalsage.github.io/posts/ctf-1/) or
[2nd part](https://crystalsage.github.io/posts/ctf-2/).

You can [register](https://tinyurl.com/2t6kwcxe) and 
[play](https://tinyurl.com/ms95pajv) this CTF. It's open to everyone.

In this 3-part series, we will see my method of approaching these challenges 
and how I solved them. The 9 challenges are divided into 3 posts.



# Decode It
![di](/ctf/di.png)

The challenge gives us a pretty clear hint that each line is mapped to _the_ single
character (The hint is ambiguous, character could either mean the person or 
a character in the flag).

And so, we are given some lines of text in a PDF file. We can quickly notice
some things. 

![di-morse](/ctf/di-morse.png)

1. There are two lines with just brackets. These are obviously the brackets
from the flag itself. We can conclude that each line somehow maps to a single
character in the flag.

2. There's no type of encoding in play here. No character/word limit for the 
line either.

3. There's no obvious pattern that stands out. Things such as skip cipher seem
out of the question.

I was a bit thrown off at first. I tried things such as measuring character
frequency to calculate any remapped letters. But then, I decided to actually
take a moment to read the content of the paragraph (Seems obvious, isn't.). 
The paragraph actually mentions Samuel Morse, the creator of Morse Code. Aha!

If we notice carefully, we see that each line contains either dits (.) or dahs
(-). Maybe we can extract all the dits and dahs from each line.

e.g. The very first line, 
> There is a town known for cryptography named Encoder-Decoder. 
> And it's not just one person skilled with cryptography, it's just 
> fully-filled with all of them.

contains 1 dah, a dot, a dah and a dot again, in that order. So, `-.-.`, 
which is clearly the letter `c`. This must be the 'c' of `csi-ctf`!

Since we know the flag format, we can actually skip the first 8 lines and 
assume them to be `csi-ctf{`. To decode everything inside the brackets, we can
extract the dits and dahs from the lines, with some simple Python.

```python
extract_symbols = lambda line: "".join([i for i in line if i in ("-", ".")])
```

Once we extract all dits and dahs, we can put them in a morse --> text converter.
I happened to have a script on my machine for this.

![di-morse-ans](/ctf/di-morse-ans.png)

Which is the text inside the brackets, so the flag is 
`csi-ctf{MORSEINTERSTELLAR}` $\blacksquare$

# Crack the Sequence
![cts](/ctf/cts.png)

The challenge description tells us that we need to crack a sequence of four
cards, and decrypt it. Cool.

If we click around on some 5 cards randomly, we get an alert box saying that
the combination is wrong. 

![ctf-fail](/ctf/cts-wrong.png)

Let's see what's going on under the hood. On the page's source,  we find the
following Javascript code.

```js
    var store = "";
    var count = 0;
    function card(res) {
      store = store + res;
      count++;
      console.log(store.length);

      if (count === 4) {
        setTimeout(() => {
          if (!(store === "d8c2436e262448af3043fde3d652726a")) {
            alert("wrong");
            window.location.reload(true);
          }
          else {
            alert("congratulations you found the flag : " + `csi-ctf{${store}}`);
            window.location.reload(true);
          }
        }, 300);
      }
    }
    function flip(event) {
      var element = event.currentTarget;
      if (element.className === "cards col-md-4 col-sm-6 col-12") {
        element.style.transform = "rotateY(180deg)";
      }
    }
```


The `card()` appends 'something' to `store`. Once we flip 5 cards, we check if
the 'something' of the cards form the sequence 
`d8c2436e262448af3043fde3d652726a`. If so, we get the flag. 

But hey!!! The 
flag is the `store` variable itself, as evident by the 
"congratulations" message. So the flag is the hex sequence
`d8c2436e262448af3043fde3d652726a`. But this doesn't make sense. If you try
submitting this, it fails. Let's try decrypting this. 



Since the total length of the hex sequence is 32, I can safely assume this 
to be the MD5 hash of the flag. We can crack this using a mask with hashcat
or a wordlist. The easiest way is to check the online databases for the
password. After checking some, I found a hit. 

<img src="/ctf/cts-decrypted.png" alt="drawing" width="500"/>

This should be our flag. `csi-ctf{sequential_flag}` $\blacksquare$

If you want to do this the proper way:
The idea is to flip the cards in correct order. We notice that the
`onClick()` events for the cards are called with specific arguments. These
seem like fragments of the entire sequence, these are the 'something'.  e.g.
we notice for the first card that `card('d8c2436e')` is called. This matches
with the first few characters of our resulting sequence.

So we collect all the `onClick` snippets.

```js
    onclick="[flip(event),card('d8c2436e')] //1
    onclick="[flip(event),card('')] //2
    onclick="[flip(event),card('')] //3
    onclick="[flip(event),card('')] //4
    onclick="[flip(event),card('262448af')] //5
    onclick="[flip(event),card('652726a')] //6
    onclick="[flip(event),card('')] //7 
    onclick="[flip(event),card('3043fde3d')] //8
```

We notice that some of these are empty. So, if we were to flip the cards in
the order `1-5-8-6`, it'll form the sequence that we want and we'll get our hash, which is the same sequence. 

![cts-flag](/ctf/cts-flag-hashed.png)


**Challenge Idea/Solution**: An alternate solution to this challenge, if the flag was something else, would be to set a breakpoint where we check if the `store` is equivalent to the sequence, and (by)pass the check entirely by modifying the `store` variable to be equal to the hash. Alternate challenge idea! 


# One Zero One
![ozo](/ctf/ozo.png)

This is easy. We just need to decode the binary. In fact, the hardest part
of this challenge is to write down the binary in a text editor. The full binary
is,

```
0100000001000011
0101001101001001
0100010001001101
0100001101000101
0100001101010100
0100011000100011
```

which decodes to, `@CSIDMCECTF#`. The challenge deviates from other challenges
to give us the text inside the brackets. I found it surprising that there
was no indication about this. The full flag is `csi-ctf{@CSIDMCECTF#}` 
$\blacksquare$

**Note**: In theory, it's much convenient to use OCR on the image to extract the
binary digits, but I could not find one reliable OCR service which could do so.
So I ended up writing them by hand.



<hr style="border:2px solid gray"> </hr>
This marks the end of the writeups.
