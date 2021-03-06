# PicoCTF_2017: SmallSign

**Category:** Cryptography
**Points:** 140
**Description:**

>This service outputs a flag if you can forge an RSA signature!
nc shell2017.picoctf.com 5596
[smallsign.py](smallsign.py)

**Hint:**

>RSA encryption (and decryption) is multiplicative.

## Write-up

If we connect to the server, we see that it gives us a RSA modulus, public exponent, and a maximum of 1 minute for it to encrypt and return any message we'd like. Once we've encrypted as many messages we'd like in our 1 minute time frame, it asks us to forge an RSA signature of a number they give.

The equations for encrypting and decrypting RSA Signatures are (assuming no padding, as is the case in this problem):

c≡mdmodNc≡mdmodN

m≡cemodNm≡cemodN

The modulii are generated securely, so theres no way we can factor those in a reasonable amount of time. But notice:

c1c2modN≡md1md2modN≡(m1m2)dmodNc1c2modN≡m1dm2dmodN≡(m1m2)dmodN

If we multiply two ciphertexts, the result of decrypting the ciphertexts will be the product of the original two messages!

So this makes our plan for forging pretty clear, we need to get the factors for our challenge encrypted in the first phase and then multiply the ciphertexts of those factors together. Except we don't know our challenge beforehand. The smaller the prime p, the more multiples of it will exist on range $[0,2^{32})$, so to increase our chances of being able to factor our random number we should get the ciphertexts of the smallest primes in the first phase. In mathematics terminology, we are looking for a challenge that is B-smooth, where B is the index of the largest prime we are looking at.

So we just have to keep on trying to get the server to sign primes with the varying modulii, until we get a challenge that is B-smooth.

@ifm-tech wrote an awesome socket connector for this problem, so handling talking to the server became easy.

Getting the server to sign our messages takes awhile, so since we can't sign that many messages in one minute, and so it will take awhile before we get a challenge which we can fully factor into our small signed primes. So we start our  [solution](https://github.com/hgarrereyn/Th3g3ntl3man-CTF-Writeups/blob/7f2daab679cd091cf45d0e375eb2a17a2a3a5f37/2017/picoCTF_2017/problems/cryptography/Small_Sign/solution.py), and go take a snack break, and then get the output:

```
$ python solution.py
We are looking at the first 168 primes
The largest prime we are obtaining is: 997
N: 23225724441260502163156858665541141018496104030765636336117495388452692752434551594356844301202757041498963343119776192971642650182341076075417853017963895654682907267507286893804382438104343728892184884725615652446029484470272738148082755208363740506711977494914732601051143901275174287629202627440351925877668239347476737619227138812585842151289661062861262965797206728938828440224425220049757239538828196446047245802293245569271831668627522474196683044076122265421666860419599298098813130818090983589816598084188896610492178986114840909079430074106751125367318111724015857433263606125379722651479325752520259889833

e: 65537

Done signing
Challenge: 1376397218

[2, 7]
Not fully factored
... <SNIP> ...
Not fully factored
Trying another iteration
N: 22222223266063880824151738523108932773683502205946922126395985656042776887111566249662284686720535925154927145294383689670743324701285415708765126660160055206471344943617192489145489058445612712791386819918849645320061889512498863436947807759584343055514570295431751957461574692165057498840925358522939303618079808372737081334870743398907529317254507082344471767854174786478680032084547254009589257992485301545754407174597193764569996342462430344753770379551006822573210288749698655245063732686224347409945959099675047318092421510183766462303784003917855223201640318019546979850657160430769416204335568349320120627599

e: 65537

Done signing
Challenge: 517194738

[2, 3, 3, 157, 197, 929]
21196551478776366791946737143923132659627817410974557628711611202905767761222602300771768239576768236096266199934553163731151613705788012484098376330512817793232983875474491249946099361975940350011595999665752717661821377952737646065850798545894003963763678218517505918274271494554382693969261463088137761054768782633205399308542037408962089614715933869756383713279518389690436376178337630369159979006126319842797929849044074933572007017462374901894191798123231314185483380674863854114106470057173760010686740585807219735365699737276761743417716992691495174132330056012134405925192803173793281342390647293692945490362
517194738
> Congrats! Here is the flag:
>  c4061d2c3315e76970f4c70e26a459cf


>
Solution obtained in 10 tries!
0.661229 seconds process time
It took 263.702187061 seconds of actual time

```

So this took a reasonable amount of time to complete!

But I want to take this one step further!

What is the optimal number of primes to use? I was guessing when I chose 1000 as the upper bound for primes in this instance. 10 tries seems to be fairly lucky from my previous tests, as I've used the primes <= 3000, and had it take 15 tries before I got the flag, with each signing period taking double the time period used for the above test case.

Let Ψ(N,B) represent the number of numbers less than N that are B-smooth  [(the de Bruijn function)](https://en.wikipedia.org/wiki/Smooth_number#Distribution), for small B we can use the approximation

Ψ(N,B)=1π(B)!∏p≤BlogNlogpΨ(N,B)=1π(B)!∏p≤Blog⁡Nlog⁡p

Note that  π(B)π(B)  is the number of primes less than or equal to B. The higher the Ψ(N,B), the more likely our challenge will be a solution.

Lets assume that the time per signing increases linearly with the number of factors (most of the time being spent in network lag)

Then we want to maximize:  Ψ(N,B)π(B)Ψ(N,B)π(B)  Through manual testing, I've determined the maximum value of B usable with my solution script, until it takes over 1 minute is around 3000, so we can just brute force for the optimal value of B!

Brute forcing for b gives the optimum B as B = 13 (including 13) Trying this though, it doesn't end up being faster. This is because the time to establish a socket takes longer than a single check. Timing by hand, the time to establish a new socket is approximately 2-3 times that of the time to check the 6 primes. So lets instead try to account for these by maximizing:  Ψ(N,B)π(B)+15Ψ(N,B)π(B)+15  (Essentially saying that those time delays are worth 2.5 checks)

Doing this, we get 17 as the optimum prime to be using. But testing, this still doesn't seem to be the case. So it looks like that the small B approximation is breaking down.

So now we have to resort to the full form of Ψ(N,B), which is equal to the  [Dickman-de-Brujin function](https://en.wikipedia.org/wiki/Dickman_function)  with a small error.

Unfortunately the Dickman-de-Brujin function involves solving the following delay differential equation.

up′(u)+p(u−1)=0up′(u)+p(u−1)=0

And well uh, this differential equation is hard. We are concerned with the range  0≤u≤10≤u≤1  which makes a unit step function hard, if not impossible to use in conjunction with the Laplace Transform. Essentially, I can't figure out how to solve this Diff. Eq. so I can't really find the optimum number of primes! But Wikipedia doesn't have the closed form solution to this Diff Eq on the Dickman function page, and in stead only has a first order approximation, so it probably is a very hard Diff. Eq.

Please let me know if you can solve this diff. eq! Then we can find the optimum number of primes to use :)

Therefore, the flag is `c4061d2c3315e76970f4c70e26a459cf`.

Thanks[[Hackademia](https://ctftime.org/team/38238)](https://hgarrereyn.gitbooks.io) for the writeupThis challenge mainly focuses on a simple concept of RSA, which is it's multiplicativity? Is that a word? Well, basically, this concept states that for a given signature of `m1`, `s1` and a signature of `m2`, `s2` would end up in the form where (`m1` * `m2`) = (`s1` * `s2`).

In this challenge, we are given the ability to make the server sign as many integers as we want for 60 seconds and then we have to also solve the challenge in that same time. This challenge is a random integer from `0` to `2**32`, which is a large number and something we cannot possibly get the server to sign to, within 60 seconds. However, we can take a shortcut and just try getting the server to sign a couple prime factors and then hoping for the RNG to get us a challenge that is factored up by the prime factors we have.

Lo and behold, [python script](solve.py) and shell loops.

    while true; do
        python3 -u smallsign.py | tee -a log
        sleep 3
    done

Therefore, the flag is `damnitiforgottosavetheflag`.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY4MTU0Mzg1NCwtMTU1NTc2MzQyOV19
-->