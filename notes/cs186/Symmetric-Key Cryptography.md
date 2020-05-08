# Symmetric-Key Cryptography

Basically, cryptography is finding a way to communicate securely over an inherently insecure communications medium. Historically, we've represented then sender and receiver of messages over this medium as *Alice* and *Bob*.

In **symmetric-key cryptography**, both Alice and Bob share the same key $K$. Here, we apply **symmetric-key encryption**, in which Alice encrypts her messages under $K$, sends a *ciphertext* to Bob, and Bob decrypts the ciphertext with key $K$. Symmetric key encryption provides **confidentiality**, in which *eavesdroppers* (Eve) cannot tell *anything* about the message Alice is sending. Generally, the key $K$ is generated randomly

**Message authentication codes** (MACs) act like a keyed checksum. It helps provide **integrity** for Alice's message, which basically immediately lets Bob know if her message has been tampered with. It also provides **authenticity**, which tells Bob that this message came from Alice for sure. We represent this MAC with $F_k(M)$, which Alice appends to her ciphertext. Since Bob has key K, he can simply recalculate $F_k(M)$ and compare it with Alice's MAC. Overall, MACs were made such that **Eve cannot mess with the message (or spoof it) without invalidating the checksum.**

**Public-key cryptography**, on the other hand, provides the CIA triad **without the need to share common keys**. The receiver, Bob generates a public key and private key. The public key is available for *anyone* to encrypt a message to send to Bob, which he will decrypt with his private key. **Digital signatures**, or public key signatures, are the public key version of MACs: they provide integrity and authenticity to sent messages via private key signing. Alice uses her *private key* to create a digital signature from her ciphertext. Alice will thus *also* generate a public, private key pair and send her *public* key to Bob, who can use it to *verify* the signature. 

Remember the goal of cryptography: ensuring the security of communications between Alice and Bob across an insecure medium, with a lurking Eve. But although we've covered Alice and Bob's roles in some detail, we still need to consider the third party in this scenario:

**How powerful is Eve?**

Remember Kerkhoff's principle: Cryptosystems should remain secure even when the attacker knows all internal details of the system. The key should be the only thing that must be kept secret, and the system should be designed to make it easy to change keys that are leaked (or suspected to be leaked). If your secrets are leaked, it is usually a lot easier to change the key than to replace every instance of the running software.

So we assume **Eve knows the encryption and decryption algorithms used**. What she *doesn't* know is the private key $K$. 

Today, we require that security ensures that we protect against some of the strongest attacks, known as **chosen-plaintext/ciphertext attacks**. In this scenario, Eve has a bunch of messages she gets Alice to *encrypt* with key $K$, and a bunch more messages that she gets Bob to *decrypt* with key $K$. If Eve can learn to decrypt any *new* messages Alice sends encrypted with key K, then Eve has broken the security of the communication. 

### One-Time Pad

The **one-time pad** is a simple encryption scheme that works like this:

Let the key $K = k_1k_2...k_n$, or be an $n$-bit secret (or private) key. We want our key to be random, so let's say bit $k_i$ is picked uniformly at random- i.e. corresponding to the $i$-th fair coin flip. Let's say Alice also wants to send an n-bit *plaintext* message $M = m_1m_2...m_n$.	

Remember what we want from our encryption scheme: 

- Map our message to a ciphertext $C = c_1c_2...c_n$
- Knowing $K$, it should be easy to decrypt $C$ to $M$.
- **Eve, who doesn't know $K$**, should NOT be able to tell *anything* about $M$. 

Encryption in the one-time-pad utilizes the $XOR$ , or ⊕, which calculates each bit of the ciphertext from the corresponding key and plaintext bits:

$$c_i = m_i ⊕ k_i$$

So we can view encryption as a function $E(M,K)$ or $E_K(M)$ that produces $C$. 

Decryption, on the other hand, calculates each bit of the *plaintext* from the corresponding key and *ciphertext* bits:

$$m_i = c_i ⊕ k_i$$

So we can view encryption as a function $D(C,K)$ or $D_K(C)$ that produces $M$.

Now comes the important part: **how much information can Eve deduce about the plaintext if she intercepts the ciphertext?** Remember that if our shit is secure, Eve should not learn any ***new*** information about $M$ (beyond what she already knew before she intercepted $C$). 

In fact, the one-time-pad, simple as it may be, satisfies this! Let's prove how: to do so, we'll consider a thought experiment whose structure will become important for proving the security of other encryption schemes.

### Proving the Security: One-Time Pad

Let's say Alice sent **one** of *two possible* messages, $M_0$ or $M_1$, to Bob. Eve wants to try and figure out which one was sent, **based on the ciphertext $C$.** **If we can prove that the probability of Eve deducing the correct message (bit) is $\frac{1}{2}$,** then we've proven she's just guessing and has gained no actual information about the plaintext. 

The probability space in this problem is $2^{n+1}$: there are $2^n$ *equally likely* possible guesses for the $n$-bit key $K$, and $2$ choices for choosing the right message Alice sent. Still assuming that $K$ was chosen at random, we also assume the message choice bit $b \in \{0,1\}$ was chosen randomly as well.  Given Eve observes the ciphertext $C$, we want to find the **conditional probability that b = 0 given her observation** (b=1 works just fine too, since the message chosen is independent of the ciphertext seen). We want to find the value of:

$$P(b=0|\text{ciphertext} =C)$$

Let's simplify this down by some rules of probability:

$$ = \frac{P(b = 0 \wedge \text{ciphertext} = C)}{P(\text{ciphertext} = C)}$$

We know the ciphertext maps directly to the choice of the key. For any *fixed* plaintext message $M$, every possible value of the ciphertext $C$ can be achieved by an appropriate and unique choice of the shared key $K$. This means that $K$ and $C$ form a bijection for a given message $M$. This also means that given the mapping between a plaintext $M$ and ciphertext $C$, we can deduce the private key $K = M ⊕ C$. Since $K$ is a random $n$-bit string, however, $C$ is as well! 

In general, **if a** **bijection exists between an input space and output space, if choice of input is uniformly random, then the choice of output is too (and vice versa).**

So Eve will just see $C$ as a **uniformly random string** of $n$ bits. 

So $P(\text{ciphertext} = C)$ is simply $(\frac{1}{2})^n$, same as $P(\text{key} = K)$. 

And since choice bit $b$ and the ciphertext $C$ are independent, $P(b = 0 \wedge \text{ciphertext} = C) = \frac{1}{2}*(\frac{1}{2})^n$.

So this simplifies down to $P(b=0|\text{ciphertext} =C) = \frac{1}{2}$ as desired! Our system is proven secure to chosen-plaintext/ciphertext attacks.

### One-Time Pad: Imperfections

Although we proved its security against our strongest threat model, this obviously does *NOT* mean this is a completely secure transmission system!

As its name suggests, this can only be used for **one** secure message transmission. If key $K$ is reused to encrypt two messages $M_1$ and $M_2$ , then Eve can XOR the two ciphertexts $C_1= M_1 ⊕ K$ and $C_2 = M_2 ⊕ K$ to obtain $C_1 ⊕ C_2 = M_1 ⊕ M_2$, which gives information about the plaintext messages (remember we didn't want Eve to find *anything* out about the plaintext from the ciphertexts). Often there is enough redundancy in messages that just knowing $M_1 ⊕ M_2$ is enough to recover most of $M_1$ and $M_2$. Soviet officials made this mistake back in the Cold War days and the US exploited. 

So overall, **if one-time pad is used to encrypt/decrypt more than once**, security is compromised. Thus in the real world, one-time pad is pretty much useless. 

In our next note, let's discuss something that's actually useful as far as symmetric encryption and decryption.

