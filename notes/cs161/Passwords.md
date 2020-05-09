# Passwords

We all know the purpose of passwords: authentication. Unfortunately, most people usually take the security of their passwords for granted. **What practices should be used to make passwords as secure as possible?**

### Risks + Weaknesses of Passwords

There's an unfortunate tradeoff in the security of a password and the ability for people to (want to) remember it. Because of this latter human trait, people tend to re-use passwords across sites as well- so if it gets compromised, everything does.

There are many possible attacks associated with password authentication:

- **Online guessing attacks**: Repeated guesses at the user’s password. The most common attack for someone who knows nothing about security- and it might actually work if the password is stupid easy. 
- **Social engineering and phishing**: An attempt to **fool** a user into revealing his password, perhaps through phishing, where we disguising oneself as trustworthy or official, e.g. a fake apple store email.
- **Eavesdropping**: Stealing an unencrypted password sent over an unencrypted web connection.  For example, if the user is connecting to the Internet over an open Wifi network, the eavesdropper can learn the user’s password.
- **Client-side malware**: If an attacker can get a keylogger or some other malware on a user's machine, the keylogger/malware can capture the user’s password and send it to the attacker. 
- **Server compromise**: Servers might contain passwords of accounts on that site. If compromised, an attacker might get these passwords and map them to accounts. 

Let's look at defenses against these five things.

### Defenses: Eavesdropping

The solution is literally given in the problem description: just encrypt the browser-server connection via SSL/TLS. This means connecting to a web site via https instead of http. 

We could also use more advanced protocols. For example, maybe a **challenge-response protocol**:

- Server sends browser a random challenge $r$ 
- Browser takes the user’s password $w$, computes $H(w, r)$ where $H$ is a cryptographic hash (e.g., SHA-256)
- Browser sends $H(w, r)$ to the server. 

The **hash** is instead sent over the server: because it's pretty much impossible to map a hash output to an input (the password in this case), the password is never actually sent over the network. However, SSL/TLS is still better. 

### Defenses: Client-Side Malware

Malware on a victim machine is extensive and attackers get incredibly creative. It's really hard to defend against all of them. 

First, keyloggers. Keyloggers are just machines that track which buttons pressed on keyboard. So they track ALL keys pressed- including when you enter your password into a login page, and send these results to the attacker. 

A proposed solution is randomized virtual keyboards: a keyboard is displayed on the screen, with the order of letters and numbers randomly permuted, and the user is asked to **click** on the characters of their password (instead of type), thus avoiding keylogger detection of entered password. However, if the malware was extended to screenshot every mouse click location, this solution is useless. 

So if you are stupid enough to get malware on your computer, your password **will** be captured. This is a fundamental flaw in passwords in themselves, leading to measures like **two-factor authentication**, where we combine the password with some other form of authentication. If you're a fellow Cal student (Go Bears), you'll have become very, very familiar with Duo Mobile and the Cal authentication portal, another example of two-factor auth. 

### Defenses: Guessing Attacks

A smart guesser will start with the most likely passwords before moving on to less likely (Or, anyone with a brain). If there are *no limits* on an attacker's guesses, he'll have a good chance of (eventually) guessing a password correctly. 

**Targeted attacks** are where the attacker wants to find a specific user's PW. **Untargeted attacks** are where the attacker just wants to guess someone's password, but doesn’t care who. What the hell is the point of an untargeted attack if you just take over some mom's minion-meme-infested facebook account? Well, one example is if the attacker wants send lots of spam from it. Then the uselessness of the account (and it being untraceable to the attacker) is important. 

We'll abstract away the math here, but in general, given the general shittiness of user passwords, it shouldn't be surprising that **resistance against untargeted attacks is very low,** and thus that spam is so common.

For targeted attacks, however, the attacker’s got to work a little harder. Of course, the chance that the attacker gets into a specific account will always depend on luck (how much the user cares about security in passwords). In general, targeted attacks are possible, but the attacker is not guaranteed a success, and it might take quite a few attempts. 

The overall solution to guessing attacks, of course: **make a secure (long) password!!**

### Lol Good One

Unfortunately there are 4 billion or so non-CS majors in the world, and most will choose a password that's memorable but easy to guess. So it's up to us to implement a login page such that we protect these idiots. 

There are some ways to do this: 

### Rate-limiting

We can limit the number of consecutive incorrect guesses that can be made, before forcing a wait time or prompting the user to enter some other authenticating information. We could also handle guessing programs by **enforcing a maximum guessing rate**; if the number of incorrect guesses exceeds, say, 5 per hour, then we take similar action as before. 

**For targeted attacks, rate-limiting is actually pretty effective**. However, it introduces the opportunity for denial-of-service attacks. If let's say Bob cheats on Alice: to get him back Alice could lock Bob's Facebook account by making a million incorrect login attempts (kind of a weak revenge, but you get the idea). In many settings, though, this denial-of-service risk is acceptable. The max-guessing rate can also *significantly* prolong the time attackers need to make a lot of guesses. 

Unfortunately, rate-limiting is not an effective defense against untargeted attacks. This comes from the fact that with millions of accounts to choose from, the guessing process is parallelized. For example, an attacker making 5 guesses for each of 200 accounts (or 1 guess against each of 1000 accounts) can expect to break into at least one of them. Here, neither form of rate-limiting will help much, since only a few attempts are used per account. 

However, rate-limiting is still by all means a good security measure to implement for websites.

### CAPTCHAs 

We could also try to limit *automated* guessing attacks. You've all *definitely* seen a CAPTCHA before: it acts a puzzle that (hopefully) only humans could solve, and thus human authentication. They also appear more frequently after an incorrect login attempt, or some arbitrary robot-detection algorithm. 

Unfortunately, this defense is not as strong as we might hope. There are black market services which will solve CAPTCHAs for you, for a fairly cheap price: $1-2 per thousand CAPTCHAs solved. 

Additionally, **CAPTCHAs also do not do much to stop untargeted attacks.** Again, because of the parallelism the attacker can employ on thousands of accounts, the number of CAPTCHAs needed to be solved is upper bounded by $O(n)$, so cost won't be too much of a problem.

## Defense: Server Compromise

A lot of websites store passwords in cleartext in its server database. This is bad, though: if the site gets hacked and the attacker gets to copy database contents, pretty much all passwords are breached. Because a lot of smart people reuse their passwords on multiple sites, *all* those accounts are compromised *per person*! Not good. 

Thus it's important that sites avoid storing passwords without some form of encryption. Unfortunately, sites don’t always follow this advice. What can we do to make password storage more secure?

### Password Hashing: Naive

A non-cleartext-password storing approach we can take is **hashing each password** and then storing the *hash* in the database. When Alice creates her account and enters her password $w$, the server will produce $H(w)$ and store $H(w)$ in the database. For future login attempts with password $w'$, server simply checks if $H(w') = H(w)$ to check for correct password. 

The one-way property of cryptographic hash functions makes it nearly impossible to recover the input (password $w$) from the output (password hash $H(w)$). Thus, if there is a security breach and the attacker steals a copy of the database, he'll only see hashes and no clear passwords- and it'll be hard to invert the hashes back to PWs.

Unfortunately, this solution isn't perfect. 

This solution falls to **offline password guessing attacks.** Suppose that Mallory hacks the website and gets the (SHA256) hash of Bob’s password. Now, offline, she can *very quickly* test guesses at Bob's password by hashing a PW guess and comparing it to Bob's PW hash. The important point here is after the initial hack and retrieval of the hashed PW, there's no need to interact with the site or any of its defenses any more: it all comes down to (smart) brute force checking offline. Additionally, the speed of offline checking is supported by the fact that SHA256 (and other crypto hashes) are generally very fast, and Mallory can test many guesses very, very rapidly (around 1 billion password hashes computable per second). 	

Another attack that can exploit this are **amortized guessing attacks**. The above attack can actually be sped up dramatically by an algorithm that avoids unnecessarily repeating work. Assume again we gain access to a bunch of password hashes in the site DB. Let's say we guess **the same  $2^{20}$ passwords for each account of a site.** Additionally, $H(w)$ is a pure hash function: same input (password), same output (password hash). We **pair the $2^{20}$ most common PWs with their hashes: $2^{20} (w, H(w))$ pairs. ** Sort this collection by hash value. Then, for each of $2^{20}$ accounts, check (via binary search) if their password hash is in the sorted list. **Pre-computing a ton of common passwords saves so much time because we only need to do this process once: everything else is just searching through the list**. 

## Improved Password Hashing

Knowing how these attacks exploit our issues, let's try a better way to hash passwords. 

One of the main problems with hashing was that the hash was a **pure function**: same inputs gave same outputs. In the amortized guessing attack, this property allows an attacker to *immediately* deduce a password from a password hash if pre-computed. 

### Salt

So let's add randomness into our hashing by *adding and storing a random salt $s$ whenever we create a new user.* **The salt only needs to be different for each user**; it doesn’t need to be secret. Then, our password hash becomes $H(w,s)$. So even if two users A and B end up with the same password, their password hashes will be different since their salts will be different: $H(w, S_A) \neq H(w, S_B)$.

So now, instead of just storing $H(w)$, we now store $s, H(w,s)$ in our database. Note that the amortized guessing attack is now prevented, because we can no longer form a pre-computed bijective mapping of common passwords to hashes! The hash is now dependent on the password *and* salt. 

Let's look at this more mathematically. Let's say that Mallory takes $n$ guesses $g_1,g_2,...,g_n$ to compute Alice's password hash, with salt $S_A$: $H(g_1, S_A), H(g_2, S_A),...,H(g_n, S_A)$. Remember that Mallory also has access to Alice's true password hash $H(w_A, S_A)$ because of her initial hack. This seems like the same pre-computing as before: what's the difference?

The difference is that with the addition of the salt argument to the password hash, Mallory has to **re-compute password hashes for a different user, because that user has a different salt!** For example, for Bob, Mallory would have to recompute $n$ *more* hashes $H(g_1, S_B), H(g_2, S_B),...,H(g_n, S_B)$.

### Slow Hash

Salting, albeit being a very good start, is not enough:  Mallory can still attempt $2^{20}$ guesses at the password against each of 100 million leaked password hashes in about a day. This is mostly because, as we stated before, crytographic hashes like SHA-256 were usually incredibly fast. 

So let's **slow this hash down**. If we had a cryptographic hash that was *purposefully* slow—say, it took 1 millisecond to compute. Then, offline PW guessing is slowed considerably, because our hash function goes from being able to calculate a billion hashes per second to only around a thousand per second. 

How can we make a hash function slower? Pretty much all practical slow hashes *iterate it*:

$H_{\text{slow}}(x) = H(H(H(...(H(x))...)))$

If $H_{\text{slow}}$ iterates $H$ $n$ times, then logically, $H_{\text{slow}}$ will be $n$ times slower than $H$.

To incorporate both slowness and salts, our fully modded hash function:

$H_{\text{slow}}(w,s) = H(H(H(...H(H(w,s),s)...)))$

And we'd store these hashed passwords along with user salt $s$. 

How much should we "tune" the slowness of the hash function to be? There reaches a point where $n$ becomes so large that the server will experience noticeable delays trying to log multiple people in. So we choose a reasonably sized $n$ for $H_{\text{slow}}$ that still keeps performance overhead in check. 

For example, let's say our site sees around 10 logins per second, i.e. pretty high-traffic. Let's choose $n$ such that evaluating $H_{\text{slow}}(w,s)$ takes 1 ms. Thus about 1% of our server's CPU power will be used in handling login traffic with this- certainly reasonable! And of course, we have the benefit of slowing offline password guessing attacks *massively* should our server/database be compromised. 

## Cryptographic Implications

Let's say we’re implementing file encryption. With encryption, we need something like a private key- a password! Unfortunately, if we just use a hashed password $H(w)$ as our symmetric key, this will easily fall prey to guessing attacks, especially if $H$ is fast. Using a slow hash might help a little, but this in reality will only increase the attack time from instantaneous to around minutes-long.

In general, if we derive a key from a common password, there is inherently not much we can do to defend against a fully resourceful and patient Mallory. **Password-based keys suck in general: random cryptographic keys, like AES-128 keys, are better** (we'd have to find a way to store this key somehow too).

## Password Alternatives



