+++
title = "CSI-CTF Writeup (Part-1)"
date=2022-02-13
description = "The 1st part of the 3-part writeup for CSI-CTF 2022"
showFullContent = false
readingTime = false
+++
I recently won 1st place at the CSI-CTF 2022. The CTF was basic, but still
amazing (We find that the two things are often mutually exclusive in many CTFs).
The challenges were easy, but not lame. The UI was fantastic. I was impressed
by the CTF infrastructure. It is not easy to host a CTF this well.

You can [register](https://tinyurl.com/2t6kwcxe) and 
[play](https://tinyurl.com/ms95pajv) this CTF. It's open to everyone.

In this 3-part series, we will see my method of approaching these challenges 
and how I solved them. The 9 challenges are divided into 3 posts.
![Some](/ctf.jpg)

A tip for solving easy CTF challenges is that the name of the challenge 
often gives a hint to you. After that, you just need to keep your mind open to all
the possibilities and think your way through. Keep the name in the back of 
your mind but don't let it bias your thinking. Keep an open approach to any
challenge which you're aiming to solve.

# What Lies Within
![challenge](/ctf/whatlieswithin.png)
From the challenge's name and some experience, we can immediately guess that the
image has data inside it. The most preliminary test you can do to weed out 
some easy challenges is to run `strings` on the image. `strings` will yield us 
the plaintext strings inside the file. 

![wlw-strings](/ctf/whatlieswithin-strings.png)

As you can see, it returns us all of the plaintext things inside the image. It
would be a mad task to scroll down and find our flag in this haystack. So, we 
can find our flag using `grep`. We can grep for the flag's format since we 
definitely know it's a part of the flag.

![flag](/ctf/whatlieswithin-flag.png)

The flag! `csi-ctf{Let's_go_to_the_subway_for_lunch}` $\blacksquare$

---

# Luminous Hunt
![lh-chall](/ctf/lh-chall.png)
In this challenge, we are again, starting with an image. And, the name gives 
us a hint about this being related to light.

My initial guess was that the image contained the flag, but it's too small to
see. I _really_ zoomed in on the image to see if there was a tiny flag somewhere
in there, but nope. Thanks creators for not doing this! 
(This is actually a pretty common thing in CTFs!)

Next, I tried putting this image in an image editor and analyzed its RGB 
channels. The flag, if  hidden in one of the three channels, is only visible
in that particular channel. But no.

I was out of my typical ideas when I enlarged the image a bit and noticed that 
the blackboard looked too black. Something seemed off about it. So, keeping the
challenge name and the weird blackboard color in mind, I decided to brighten
up the image, and play around with the contrast, which gives us our flag, 
hidden in the darkness of the blackboard.

![lh-brighten](/ctf/lh-brighten.png)

It's still somewhat hard to read, but we can make out `csi-ctf{CPL-CSI}`, which
is our flag. $\blacksquare$

---

# A Tribute
![at-chall](/ctf/at-chall.png)
We are given a GitHub page which tells us to find the name of a person. This
obviously is an OSINT challenge, right? No! There's a twist here.

If you find out the person's name, and try to submit it as a flag, the 
submission gets denied.

If you take a look around the page, you'll see that the repository's commit
history is pretty interesting. It contains some strings that could be useful 
to us.
![at-commits](/ctf/at-commits.png)
The letters look jumbled but they are still readable. This is probably encoded
with some cipher. If you've had any sort of experience with ciphers, you've
probably heard of the Caesar cipher, AKA rot-n cipher. One of the strings of
the commit messages must be our encoded flag. 

If you patiently try out some/all of these, you'll find out that the 4th commit 
message from the top is actually our flag, with rot-18 encoding.
![at-flag](/ctf/at-flag.png)

The flag is `csi-ctf{congratulations_the_flag_is_jira}` $\blacksquare$




<hr style="border:2px solid gray"> </hr>
This series is to be continued in the part 2 and 3 where we'll solve more
challenges...
