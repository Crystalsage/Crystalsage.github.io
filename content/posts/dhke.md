+++
title = "A case of parameter injection MiTM on DHKE"
date=2022-05-19
description = "Documenting a small varient of MiTM attcak on Diffie-HellmanKey Exchange."
showFullContent = false
readingTime = false
+++

You can read up on Diffie-Hellman Key Exchange in various articles, books or get started on [Wikipedia](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). We’ll begin the discourse with a short intro anyway.

# Background
In DH key exchange scheme, two interested parties Alice and Bob participate in an exchange, at the end of which both have a shared secret. This secret is jointly computed over their inputs, one of which is kept private.

Let the two parties be Alice and Bob. Two global parameters $g$
and $p$ are decided upon at the start of the exchange. $g$ is the generator of the group $(\mathbb{Z}/p\mathbb{Z})^\times$. $p$ is a prime integer. $g$ is then said to be the primitive root modulo $p$.

Once the exchange begins,

1. Alice chooses an integer $a$, which is her private key. Alice then calculates $A \equiv g^a \pmod{p}$, which is declared as her public key, and is sent to Bob.

2. Bob does the same and chooses b , his private key and calculates $B \equiv g^b \pmod{p}$, his public key. This is sent to Alice.

3. The shared secret $S$ is computed by Alice as $S \equiv B^a \pmod{p}$ and by Bob as $S \equiv A^b \pmod{p}$. Both values are found to be equal.

<div style="padding: 0.5rem;border-top:2px solid white;border-bottom:2px solid white;border-left:2px solid white;border-right:2px solid white;" >
<b style="color: #78e2a0" >Remember</b><br><br>
$X \equiv g^x \pmod{p}$ is hard to reverse because of discrete logarithm problem.
<p>$A$, $B$, $g$ and $p$ are public. $a$ and $b$ are kept private by the respective parties.</p>
</div>

<h1 id="the-challenge">The challenge<a href="#the-challenge" class="hanchor" ariaLabel="Anchor">&#8983;</a> </h1>
<p>The challenge is served to us through a Python script. I&rsquo;ve snipped
away some part of it to maintain focus and left some comments. Take a read.</p>
<div class="highlight"><pre tabindex="0" style="color:#f8f8f2;background-color:#272822;-moz-tab-size:4;-o-tab-size:4;tab-size:4;"><code class="language-python" data-lang="python"><span style="display:flex;"><span><span style="color:#75715e"># The flag is a dummy one. The real flag is set on the server.</span>
</span></span><span style="display:flex;"><span>FLAG <span style="color:#f92672">=</span> <span style="color:#e6db74">&#34;CTF{--REDACTED--}&#34;</span>
</span></span><span style="display:flex;"><span>DEBUG_MSG <span style="color:#f92672">=</span> <span style="color:#e6db74">&#34;DEBUG MSG - &#34;</span>
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span><span style="color:#75715e"># The prime and base</span>
</span></span><span style="display:flex;"><span>p <span style="color:#f92672">=</span> <span style="color:#ae81ff">0x509efab16c5e2772fa00fc180766b6e62c09bdbd65637793c70b6094f6a7bb8189172685d2bddf87564fe2a6bc596ce28867fd7bbc300fd241b8e3348df6a0b076a0b438824517e0a87c38946fa69511f4201505fca11bc08f257e7a4bb009b4f16b34b3c15ec63c55a9dac306f4daa6f4e8b31ae700eba47766d0d907e2b9633a957f19398151111a879563cbe719ddb4a4078dd4ba42ebbf15203d75a4ed3dcd126cb86937222d2ee8bddc973df44435f3f9335f062b7b68c3da300e88bf1013847af1203402a3147b6f7ddab422d29d56fc7dcb8ad7297b04ccc52f7bc5fdd90bf9e36d01902e0e16aa4c387294c1605c6859b40dad12ae28fdfd3250a2e9</span>
</span></span><span style="display:flex;"><span>g <span style="color:#f92672">=</span> <span style="color:#ae81ff">2</span>
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span><span style="color:#75715e"># Decrypt with AES. </span>
</span></span><span style="display:flex;"><span><span style="color:#66d9ef">def</span> <span style="color:#a6e22e">decrypt</span>(encrypted, shared_secret):
</span></span><span style="display:flex;"><span>    key <span style="color:#f92672">=</span> hashlib<span style="color:#f92672">.</span>md5(long_to_bytes(shared_secret))<span style="color:#f92672">.</span>digest()
</span></span><span style="display:flex;"><span>    cipher <span style="color:#f92672">=</span> AES<span style="color:#f92672">.</span>new(key, AES<span style="color:#f92672">.</span>MODE_ECB)
</span></span><span style="display:flex;"><span>    message <span style="color:#f92672">=</span> cipher<span style="color:#f92672">.</span>decrypt(encrypted)
</span></span><span style="display:flex;"><span>    <span style="color:#66d9ef">return</span> message
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span><span style="color:#66d9ef">def</span> <span style="color:#a6e22e">main</span>(s):
</span></span><span style="display:flex;"><span>	<span style="color:#75715e"># Server calculates its keys</span>
</span></span><span style="display:flex;"><span>    c <span style="color:#f92672">=</span> random<span style="color:#f92672">.</span>randrange(<span style="color:#ae81ff">2</span>, p <span style="color:#f92672">-</span> <span style="color:#ae81ff">1</span>)
</span></span><span style="display:flex;"><span>    C <span style="color:#f92672">=</span> pow(g, c, p)
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>	<span style="color:#75715e"># Enter our public key here</span>
</span></span><span style="display:flex;"><span>    M <span style="color:#f92672">=</span> receiveMessage(s, <span style="color:#e6db74">&#34;Enter The Public Key of The Memory: &#34;</span>)
</span></span><span style="display:flex;"><span>    shared_secret <span style="color:#f92672">=</span> pow(M, c, p)
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>    encrypted_sequence <span style="color:#f92672">=</span> receiveMessage(s, <span style="color:#e6db74">&#34;Enter The Encrypted Initialization Sequence: &#34;</span>)
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>	<span style="color:#75715e"># The cipher text should be a hex digest</span>
</span></span><span style="display:flex;"><span>    <span style="color:#66d9ef">try</span>:
</span></span><span style="display:flex;"><span>        encrypted_sequence <span style="color:#f92672">=</span> bytes<span style="color:#f92672">.</span>fromhex(encrypted_sequence)
</span></span><span style="display:flex;"><span>        <span style="color:#66d9ef">assert</span> len(encrypted_sequence) <span style="color:#f92672">%</span> <span style="color:#ae81ff">16</span> <span style="color:#f92672">==</span> <span style="color:#ae81ff">0</span>
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>    sequence <span style="color:#f92672">=</span> decrypt(encrypted_sequence, shared_secret)
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>    <span style="color:#66d9ef">if</span> sequence <span style="color:#f92672">==</span> <span style="color:#e6db74">b</span><span style="color:#e6db74">&#34;Initialization Sequence - Code 0&#34;</span>:
</span></span><span style="display:flex;"><span>        sendMessage(s, <span style="color:#e6db74">&#34;</span><span style="color:#ae81ff">\n</span><span style="color:#e6db74">&#34;</span> <span style="color:#f92672">+</span> DEBUG_MSG <span style="color:#f92672">+</span> <span style="color:#e6db74">&#34;Reseting The Protocol With The New Shared Key</span><span style="color:#ae81ff">\n</span><span style="color:#e6db74">&#34;</span>)
</span></span><span style="display:flex;"><span>        sendMessage(s, DEBUG_MSG <span style="color:#f92672">+</span> <span style="color:#e6db74">f</span><span style="color:#e6db74">&#34;</span><span style="color:#e6db74">{</span>FLAG<span style="color:#e6db74">}</span><span style="color:#e6db74">&#34;</span>)
</span></span><span style="display:flex;"><span>    <span style="color:#66d9ef">else</span>:
</span></span><span style="display:flex;"><span>        exit()
</span></span></code></pre></div><p>So we know the plaintext! We need to encrypt the string <code>Initialization Sequence - Code 0</code> such that it decrypts correctly. But, to our dismay, we find that we <strong>don&rsquo;t know the shared secret</strong> and there&rsquo;s no way to derive it since we never received server&rsquo;s public key $C$. How can we hope to find the
shared secret then?</p>
<div style="padding: 0.5rem;border-top:2px solid white;border-bottom:2px solid white;border-left:2px solid white;border-right:2px solid white;" >
<b style="color: #78e2a0"><u>Halt</u></b><br><br>
Ponder over the problem and try to figure out the flaw in the script before heading further. (Hint: It's somewhere in `main()`)
</div>
<h1 id="the-solution">The solution<a href="#the-solution" class="hanchor" ariaLabel="Anchor">&#8983;</a> </h1>
<p>We don&rsquo;t need to <em>derive</em> the shared secret if we can just make it
predictable! The way to do this would be to send a &lsquo;bad&rsquo; public key.</p>
<p>If you see how the shared secret is being derived,</p>
<div class="highlight"><pre tabindex="0" style="color:#f8f8f2;background-color:#272822;-moz-tab-size:4;-o-tab-size:4;tab-size:4;"><code class="language-python" data-lang="python"><span style="display:flex;"><span>shared_secret <span style="color:#f92672">=</span> pow(M,c,p)
</span></span></code></pre></div><p>you&rsquo;ll notice that our public key is used as the <em>base</em> here, which means
we have complete control over <a href="https://en.wikipedia.org/wiki/Group_(mathematics)">group</a> that is being formed. So, if
[
M \equiv 0 \pmod{p}
]
then</p>
<p>[
S \equiv M^{b} \equiv 0 \pmod{p}
]</p>
<p>And we made $S$ predictable! This is because powers of $M$ are also
congruent to 0 mod p.</p>
<p>This means we can send in a ciphertext
encrypted with the key &lsquo;0&rsquo; and it should decrypt correctly!</p>
<div class="highlight"><pre tabindex="0" style="color:#f8f8f2;background-color:#272822;-moz-tab-size:4;-o-tab-size:4;tab-size:4;"><code class="language-python" data-lang="python"><span style="display:flex;"><span><span style="color:#f92672">from</span> Crypto.Cipher <span style="color:#f92672">import</span> AES
</span></span><span style="display:flex;"><span><span style="color:#f92672">from</span> Crypto.Util.number <span style="color:#f92672">import</span> long_to_bytes
</span></span><span style="display:flex;"><span><span style="color:#f92672">import</span> hashlib
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span><span style="color:#75715e"># Send in 0 as our key (shared secret)</span>
</span></span><span style="display:flex;"><span>key <span style="color:#f92672">=</span> hashlib<span style="color:#f92672">.</span>md5(long_to_bytes(<span style="color:#ae81ff">0</span>))<span style="color:#f92672">.</span>digest()
</span></span><span style="display:flex;"><span>cipher <span style="color:#f92672">=</span> AES<span style="color:#f92672">.</span>new(key, AES<span style="color:#f92672">.</span>MODE_ECB)
</span></span><span style="display:flex;"><span>message <span style="color:#f92672">=</span> cipher<span style="color:#f92672">.</span>encrypt(<span style="color:#e6db74">b</span><span style="color:#e6db74">&#34;Initialization Sequence - Code 0&#34;</span>)
</span></span><span style="display:flex;"><span>print(message<span style="color:#f92672">.</span>hex())
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span>
</span></span><span style="display:flex;"><span><span style="color:#75715e"># Returns the following digest. This is our encrypted text.</span>
</span></span><span style="display:flex;"><span><span style="color:#75715e"># 1af761314a07bf79f31aeb53bc9e1335e1749e1142b326d82a3c29ac37a042bf</span>
</span></span></code></pre></div><p>Once we submit the ciphertext to the server, we get our flag! Notice that we entered our public key $M$ as 0.</p>
<p><img src="/conn.png" alt="di"></p>
<h1 id="key-takeaway">Key takeaway<a href="#key-takeaway" class="hanchor" ariaLabel="Anchor">&#8983;</a> </h1>
<p><em>(Pun intended)</em></p>
<p>While the order of operations does not matter mathematically, i.e.</p>
<blockquote>
<p>$S = A^b = (g^a)^b = (g^b)^a = B^a \pmod{p}$</p>
</blockquote>
<p>It clearly did matter to us. This vulnerability was only possible because
we had control over the base (And subsequently, the group that was formed).</p>
<p><strong>Note</strong>: Instead of 0, we could have also chosen a value of $M$ that is divisible by $p$, and it would have meant the same thing. (Why?)</p>
<h1 id="exercise-to-the-reader">Exercise to the reader<a href="#exercise-to-the-reader" class="hanchor" ariaLabel="Anchor">&#8983;</a> </h1>
<ul>
<li>
<p>The full challenge is here: <a href="https://gist.github.com/CrystalSage/910c26588dc8337bf8485d93209e0a5f">Code</a>. Try solving the challenge yourself by retracing my steps! :)</p>
</li>
<li>
<p>The reader is invited to solve a similar Cryptopals challenge based on parameter injection: <a href="https://cryptopals.com/sets/5/challenges/34">Challenge</a>.</p>
</li>
<li>
<p>What&rsquo;s the group that we formed?</p>
</li>
</ul>
<p>[\blacksquare\blacksquare\blacksquare]</p>

