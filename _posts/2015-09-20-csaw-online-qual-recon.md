---
layout: post
title: CSAW CTF Qualification Round 2015 - Recon
---

## Alexander Taylor - 100

http://fuzyll.com/csaw2015/start

`CSAW 2015 FUZYLL RECON PART 1 OF ?: Oh, good, you can use HTTP! The next part is at /csaw2015/<the acronym for my university's hacking club>.`

If we go to Alexander Taylor's Linkedin profile, https://www.linkedin.com/in/fuzyll you will see that he is the president of Whitehatters Computer Security Club, who's acronym is appended to the URI is:

http://fuzyll.com/csaw2015/wcsc

`CSAW 2015 FUZYLL RECON PART 2 OF ?: TmljZSB3b3JrISBUaGUgbmV4dCBwYXJ0IGlzIGF0IC9jc2F3MjAxNS88bXkgc3VwZXIgc21hc2ggYnJvdGhlcnMgbWFpbj4uCg==`

Base64 decoding the above string gives us: `Nice work! The next part is at /csaw2015/<my super smash brothers main>.`

If we do a quick search with Alexander Taylor's nickname, Fuzyll and Super Mario then we find that his nickname is linked with yoshi: http://fuzyll.com/csaw2015/yoshi

We see a picture of yoshi and if we view it in notepad++, we will see the following string.

`SAW 2015 FUZYLL RECON PART 3 OF ?: Isn't Yoshi the best?! The next egg in your hunt can be found at /csaw2015/<the cryptosystem I had to break in my first defcon qualifier>`

http://fuzyll.com/csaw2015/enigma

`CSAW 2015 FUZYLL RECON PART 4 OF 5: Okay, okay. This isn't Engima, but the next location was "encrypted" with the JavaScript below: Pla$ja|p$wpkt$kj$}kqv$uqawp$mw>$+gwes6451+pla}[waa[ia[vkhhmj`

```
var s = "Pla$ja|p$wpkt$kj$}kqv$uqawp$mw>$+gwes6451+pla}[waa[ia[vkhhmj"
var c = ""
for (i = 0; i < s.length; i++) {
    c += String.fromCharCode((s[i]).charCodeAt(0) ^ 0x4);
}

console.log(c);
```

You will get back: `The next stop on your quest is: /csaw2015/they_see_me_rollin`: http://fuzyll.com/csaw2015/they_see_me_rollin

`CSAW 2015 FUZYLL RECON PART 5 OF 5: Congratulations! Here's your flag{I_S3ARCH3D_HI6H_4ND_L0W_4ND_4LL_I_F0UND_W4S_TH1S_L0USY_FL4G}!`

## Julian Cohen - 100

https://twitter.com/HockeyInJune/status/641716034068684800

`flag{f7da7636727524d8681ab0d2a072d663}`

## Eric Liang

Unfortunately our team did not finish this challenge, it appears that that they were looking to lead us with hints to: https://ctf.isis.poly.edu/static/archives/2013/competitors/index.html


