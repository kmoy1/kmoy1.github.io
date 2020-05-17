# Final: RunThrough 1

**Q1 (T/F): Web browsers protect against clickjacking via SOP, which prevents sites from putting other origins into an iframe.** 

**A1: False. Clickjacking *specifically* bypasses SOP.**

Remember that a **frame** embeds another document within an HTML doc, and is specified by the <iframe> tag. Any site can frame another site. Usually, SOP prevents a malicious site like evil.com from using scripts to get data from, say, citibank.com.

We also know that clickjacking **bypasses** the SOP for frames. In it, evil.com frames good site (citibank), and *hides* this frame by putting dialogue boxes or other elements on top of parts of framed site to create a different effect. Overall, the point is we make the user think he's clicking something when in reality he clicks something else!

**Q2 (T/F): The primary danger of XSS vulnerabilities is that they let an attacker execute Javascript on the victim machine without having the victim visit the attacker’s website.** 

**A2: False.** Although browser execution of Javascript is certainly integral to XSS attacks, the *primary* danger of XSS is it bypasses SOP.  It's already pretty easy to run Javascript on a browser through stuff like ads and analytics.

**Q3 (T/F) Even if you carefully inspect all links that you click, you can still be vulnerable to a CSRF attack**. 

**A3: True**. Hidden forms and specially-crafted image tags are common examples of non-URL vectors that can be exploited and trick user into submitting an unintended web request.

**Q4 (T/F): For most common implementations of session cookies (as seen in lecture and on the project), a SQL injection can let an attacker steal the sessions of other users.**

**A4: True.** Most session cookies are stored in the server's backing database, which SQL injection does work on.

**Q5: If a script is loaded from another origin using a script tag, the same-origin policy prevents this script from reading the cookies on the current page**

**A5: False.**

Javascript runs with the loader's origin. So same origin policy wouldn't prevent this. However, if cookies are set to HTTP-only, Javascript wouldn't be able to read this. 

## Combining Encryption Schemes

Let's say we have two symmetric encryption schemes, SymEncA and SymEncB. One might be insecure. 

Let's say Bob wants to *combine* two schemes, hoping to fix this security. He proposes the following combinational construction.

Construction 1: Concatenate SymEncA and SymEncB's encryption on $M$ to form ciphertext

1. Create $C_1 =$ SymEncA.Encrypt(k;M)
2. Create $C_2 =$ SymEncB.Encrypt(k;M)
3. Create ciphertext $C = (C_1, C_2)$.

**Q1: If *at least* one of the encryption schemes is secure**, is this scheme secure? 

**A1: ** Absolutely not. First of all, this is stupid because we've got 2 (encrypted) copies of a message in that ciphertext. And also say SymEncA's "encryption" just did nothing. Then $C_1$ would just display the entire plaintext. 

Construction 2: Chaining

1. Create intermediate $I = \text{SymEncA.Encrypt}(k,M)$.
2. Create final ciphertext $C = \text{SymEncB.Encrypt}(k,I) = \text{SymEncB.Encrypt}(k,\text{SymEncA.Encrypt}(k,M))$.

**Q2: If *at least* one of the encryption schemes is secure**, is this construction 2 secure? 

**A1:  Better than construction 1, but no.** 

One example is if the encryption scheme simply leaks the key: i.e. $\text{SymEncB(k,M) = (k,M)}$. Then everything is fucked- if an adversary ever finds secret key everything is compromised no matter what. 

Another more subtle example is if both encryptions are CTR, but with a small difference in using the counter.

Let's say SymEncA produced:

$C_0 = IV$

$C_i = \text{AES}_k(IV+i) ⊕ M$

And SymEncB produced: 

$C_0 = IV$

$C_i = \text{AES}_k(IV+i-1) ⊕ M$

Then $C_1 = AES_k(IV) ⊕ (\text{AES}_k(IV+1) ⊕ M_1)$

## Low-level Denial of Service

Let's help Mallory develop new ways to conduct denial-of-service (DoS) attacks.

CHARGEN and ECHO are services provided by some UNIX servers. For every **UDP** packet arriving at port 19, CHARGEN sends back a random packet, 0-512 chars. For every UDP packet arriving at port 7, ECHO sends back that packet (same content).

Let's say Mallory wants to DoS the two servers. One server with IP address A supports CHARGEN, and another server with IP address B supports ECHO. Mallory can spoof IP addresses.