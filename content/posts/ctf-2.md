+++
title = "CSI-CTF Writeup (Part-2)"
date=2022-02-20
description = "The 2nd part of the 3-part writeup for CSI-CTF 2022"
showFullContent = false
readingTime = false
+++

This is the part 2 of the 3-part series for the CSI-CTF writeup. Click 
[Here](https://crystalsage.github.io/posts/ctf-1/) to read the 1st part.

You can [register](https://tinyurl.com/2t6kwcxe) and 
[play](https://tinyurl.com/ms95pajv) this CTF. It's open to everyone.

In this 3-part series, we will see my method of approaching these challenges 
and how I solved them. The 9 challenges are divided into 3 posts.

# Attack on Bank
![here](/ctf/AOB.png)

The challenge description gives us info about the encrypted text (Also notice
how ECB abbreviation gives us a hint about 128-bit AES-ECB being the cryptosystem).
We also get an unique ID for the bank, which should be the decryption key, if my guess is correct. 

The link given in the challenge leads us to a Paste(bin) where we can find some encrypted text, encoded in base64: `Keg7B3p7TENwcvdk2zsy+NXVQzMnMxg6ik2xO0M6A3c=`

Cool. This is easy. We can use what we assume to be the key as the key, and
decrypt the text. I used my favorite tool, [CyberChef](https://gchq.github.io/CyberChef/) 
for this.

![aob-flag](/ctf/aob-flag.png)

We first decoded the base64 encoded plaintext, and decrypted the text using the
key, and a blank IV. This gets us our flag! `csi-ctf{pR0videncE_$4ys_He110}` $\blacksquare$

# JS Nerds
![jsn](/ctf/jsn.png)

We get login page right off the bat. I was initially thrown off by the obvious
hints towards a Blind SQL injection. _You can't see me_ is really
suggestive. To test this if there's any SQLi involved at all, 
I used typical payloads such as `' OR 1=1 --` and friends. 
But this yielded nothing even after 5-6 minutes of trying, so I eventually gave up on this.

If we visit this page's source, we can find a file at the bottom named 
`index.js` referenced in there. This `index.js` file contains some Javascript
code.

```js
function checkData(event) {
    console.log("click");
    event.preventDefault();
    let x = document.forms["login-form"]["username"].value;
    const p = document.forms["login-form"]["password"].value;
    // oh no! Client side validation always fails! thank god I have encrypted it!
    if (x === "John" || SHA1(p) == "b89356ff6151527e89c4f3e3d30c8e6586c63962") {;
    } else {
        alert("Invalid username or password");
    }

};
```

Oh. We get the username and SHA-1 hash of our password. So I guess we have
to crack SHA-1 now. We can do this using a tool such as hashcat and crack this
password using a wordlist. But I didn't happen to have my machine with me,
so I had to use [crackstation](https://crackstation.net/) to do this. Fortunately,
crackstation had this hash in their database, which reveals the password to be
`adminz`.

![jsn-hash](/ctf/jsn-hash.png)

We can log in to the site with our credentials now to get the flag.

![jsn-flag](/ctf/jsn-flag.png) 

Bread! `csi-ctf{eXploIted_wEb}` $\blacksquare$


# Computer Engineering
![ce-1](/ctf/ce1.png)

We get another Github page. Except this time, there is nothing in
the 6 commits. We do notice the 6 branches though.

![ce-2](/ctf/ce2.png)

There are 6 branches, let's visit CS first. There are 3 commits in
there, one of which gives us the flag. 

![ce-3](/ctf/ce3.png)

Flag! `csi-ctf{this_is_the_flag_codecocomo}` $\blacksquare$

This challenge was really more of a "click 50 times to get the flag" challenge.
These types of challenges where flags are hidden in commit histories and 
branches are not included **in CTFs** due to them having less of a problem
solving aspect to them.

**However**, every often, some company get pwned or a bug hunter gets rewarded
handsomely because he found an internal API key or SSH key in a Github 
commit history, which was accidentally pushed to the repo. So, it's not 
unrealistic at all. There's a lesson to be learned here.

<hr style="border:2px solid gray"> </hr>
This series is to be continued in part 3 where we'll solve more
challenges...
