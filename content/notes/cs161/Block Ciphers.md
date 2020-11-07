# Symmetric-Key Cryptography: Block Ciphers

The block cipher is a fundamental block (nice) in implementing a secure symmetric encryption scheme.

Remember that our encryption scheme used a $k$-bit key $K$ (k just to distinguish it from the number of message bits) to encrypt an $n$ - bit plaintext into an $n$ bit ciphertext. We can represent $E(M, K)$ in mathematical notation:

$$E: \{0,1\}^k \times \{0,1\}^n \rightarrow \{0,1\}^n$$

Or, after fixing key $K$ (creating $E_K(M)$):

$$E_K: \{0,1\}^n \rightarrow \{0,1\}^n$$ 

The standard block cipher $E_K$ for our purposes is the **Advanced Encryption Standard (AES)**. Usually, AES uses a block length of $n = 128$ bits and a key length of $k = 128$ bits. It was improved over the years to be both fast and incredibly secure, such that the best possible attack is *exhaustive key search*, trying all possible keys on a given ciphertext to see which returns a sensible plaintext. This means that with 128 bit keys, the attackers would have to (on average) try $2^{127}$ keys before getting something- basically completely impossible even for supercomputers. 

So AES, fittingly, is the standard block cipher encryption and decryption: even with a ton of plaintext/ciphertext pairs, there appears to be no effective way to decrypt any new ciphertexts. 

Here is an important fact about $E_K$ (and block ciphers): **to be truly secure, it needs to be a (invertible/bijective) permutation of the $n$-bit input string,** since decryption needs to reverse the action done by encryption. Additionally, the permutation must be determined solely by key $K$, and thus appear completely random to Eve. 

Like the one-time-pad, we want a measure of security of the block cipher, or at least be able to prove its security in any given scenario. Let's use a thought experiment again!

First, we give Eve one of two possible boxes, which we choose for her at random and **don't tell her which**. 

- A box that contains the true encryption function $E_K$ with a random key $K$.
- A permutation function $\pi(x)$ that permutes an $n$-bit input $x$. The permutation function is chosen uniformly at random as well, if this box is given to Eve.

Eve is allowed $T$ steps in which to "play" with the box, i.e. produce $T$ input-output pairs for whatever input she wants. So if she gets a type-1 box, she'll be getting $y_i = E_K(x_i)$, and for type-2 boxes, she'll get $y_i = \pi(x_i)$, for $i$ from 1 to $T$. 

Now, Eve must guess whether the box is type-1 or type-2. If she can't, then block cipher $E_K$ is in effect indistinguishable from a **random permutation** on its input, i.e. we've proven that it's secure!

We can measure the **advantage of Eve** in this scenario through the Adv(Eve) = $2|p − \frac{1}{2}|$, where $p$ is the probability Eve guesses the box correctly. NOTE: In this case, as with the guessing of Alice's message bit in the one-time-pad proof, $p$ is lower-bounded by $\frac{1}{2}$ and upper bounded by $1$. 

We'd ideally like to **upper-bound the advantage of Eve** by some arbitrary constant $\epsilon \in [0,1]$. If Adv(Eve) $\le \epsilon$ then we say that the block cipher is $(T,\epsilon)$-secure.

If Eve wants Adv(Eve) = 1, then box guess probability will have to be 1, and she'll have to make $T = 2^{127}$ guesses on average, as discussed above. Because of a general tradeoff between T and $\epsilon$, some sense $\log (\frac{T}{\epsilon})$ is the “effective key length” of the block cipher, i.e. a measure of how much work Eve must exert as equated with brute-forcing a key of the effective length.  For example, even if Eve can try $T = 2^{64}$ steps, a massive amount of computation, advantage $\epsilon$ is upper bounded by $\frac{1}{2^{64}}$, which means she's learning pretty much nothing! 

## Symmetric Encryption with Block Ciphers

So now that we've shamelessly slobbered over the security dominance of block ciphers, AES in particular, let's utilize it to build some secure symmetric encryption schemes.

We'd like to encrypt **arbitrarily** long messages using a fixed-length block cipher. This means we'll extend our general symmetric encryption scenario to unbound the bits allowed for plaintext and ciphertext (same for decryption):

$$E_K: \{0,1\}^* \rightarrow \{0,1\}^*$$

$$D_K: \{0,1\}^* \rightarrow \{0,1\}^*$$

Additionally, we want to be able to send repeated messages securely, i.e. ensure that **identical plaintexts produce different ciphertexts**. To do this, the encryption algorithm needs to be non-constant in some way. It can either be **randomized**, i.e. flipping coins during execution, or be **stateful**, which means that its execution depends on some (random) state information. The decryption algorithm must always map a ciphertext back to its producing plaintext, so it must be pure. 

Let's go over the big 4 block cipher symmetric encryption algorithms: 

### ECB Mode (Electronic Code Book)

In ECB mode the $l*n$-bit plaintext $M$ is simply broken into $l$ $n$-bit blocks $M_1,M_2,...M_l$. Each block is passed through the cipher to create $l$ ciphertext blocks, block $i$ denoted $C_i = E_K(M_i)$. Ciphertext simply concatenates these blocks, which pass through decryption blocks. 

If you didn't tell already, **ECB mode is ass.** For example, if a ciphertext block is redundant, i.e. $C_i = C_j$, we will immediately know that $M_i = M_j$: remember we don't want to be able to tell *anything* about plaintexts from ciphertexts! 

### CBC Mode (Cipher Block Chaining)

In CBC mode, we generate a random $n$-bit **input vector**, (IV), for a plaintext message $M$. 

We append the ciphertext with this IV (essentially sending it to Bob), denoted as $C_0 = IV$. 

Then, future ciphertext blocks are formed in a **chaining process**, each ciphertext block utilizing the previously produced ciphertext block along with the corresponding message block:

$$C_i = E_K(C_{i−1} ⊕ M_i). $$

This means that with $l$ message blocks we'll have $l+1$ ciphertext blocks, the first block being the $IV$. We concatenate these $l+1$ ciphertext blocks to form ciphertext $C = IV||C_1||C_2||...C_1$. 

Assuming a secure block cipher like AES, CBC mode has been proven to be fairly secure in terms of plaintext message confidentiality.

### OFB (Output Feedback)

Again assuming $l$ plaintext blocks of $M$, in OFB mode, we first generate our $n$-bit $IV$. Then, we **repeatedly encrypt** to obtain a set of values $Z_i$ as follows: 

$$ Z_0 = IV $$

$$ Z_i = E_K(Z_{i−1})$$

To correspond with $l$ plaintext blocks, we'll produce $Z_0$ to $Z_{l-1}$: $l$ total values. Each of these values will now be treated as a **key** for one-time pads on each plaintext block:

$$C_i = Z_i ⊕ M_i$$

Like CBC mode, overall ciphertext simply concatenates the IV and the blocks.

Unfortunately, this has its imperfections as well, particularly with **ciphertext tampering**. For example, let's say that Eve, in addition to knowing **all** ciphertext blocks (as usual), also knew a block of plaintext $M_i$. He can simply apply $C_i ⊕ M_i = Z_i$. Now having this block key, if Eve wants, she can replace plaintext block $M_i$ with whatever she wants and retain the integrity of the other ciphertext blocks, exploiting the parallel nature of this scheme. This might be really bad if plaintext block $M_i$ is a command to mutate some sensitive information like money transfer amount and recipient. 

### CTR Mode (Counter)

Finally, we have counter mode. Note that in CBC mode, ciphertext blocks were dependent on the ciphertext block created before it: this means that encryption had to be sequential as opposed to parallel (OFB/ECB). Parallelizing processes usually means we can speed it up way more (but we still must retain security!!). 

We'll utilize a **counter** in this mode, initialized to IV. For each plaintext message block, we **increment this counter**, then encrypt it to serve as a OTP key (like OFB). So we have:

$$Z_i = E_K(IV + i) $$

$$C_i = Z_i ⊕ M_i$$

