# RSA Broadcast Attacks: Håstad, Coppersmith & Miller–Rabin

## 📌 Topic

This project studies the security of **RSA encryption in broadcast scenarios**, with a particular focus on the vulnerabilities that arise when the **same plaintext is encrypted for multiple recipients using the same public exponent**.

The work combines theoretical analysis and experimental simulations to investigate:

- RSA key and prime generation using the **Miller–Rabin primality test**
- The classical **Håstad Broadcast Attack**
- Integer root extraction using **Newton's method and binary search**
- The extension of Håstad's attack to **linearly padded messages**
- **Coppersmith's small-root method**
- **Lattice construction and LLL reduction**
- The security implications of deterministic versus randomized RSA padding

---

## 🔹 Part I — Miller–Rabin & RSA Prime Generation

The first part investigates the generation of large prime numbers required for RSA.

The **Miller–Rabin probabilistic primality test** is implemented and analyzed theoretically and experimentally. The study examines the relationship between the number of iterations and the probability of incorrectly identifying a composite number as prime.

The experiments also evaluate the number of random candidates required to generate primes of different bit lengths and compare the empirical results with theoretical predictions from the **Prime Number Theorem**.

### Key Finding

The experimental results confirm the theoretical error bound of Miller–Rabin:

$$
P(\text{error}) \leq 4^{-t}
$$

and show that the average number of candidates tested before finding an $\ell$-bit prime follows the expected theoretical behavior.

---

## 🔹 Part II — Håstad's Broadcast Attack

The second part studies the **RSA broadcast attack**.

When the same message `m` is encrypted for several recipients using the same small public exponent `e` but different RSA moduli:

$$
c_i \equiv m^e \pmod{N_i},
$$

an attacker can use the **Chinese Remainder Theorem (CRT)** to reconstruct the value of $m^e$ modulo the product of the RSA moduli.

If:

$$
m^e < \prod_{i=1}^{r}N_i,
$$

the reconstructed value is exactly $m^e$, meaning that the attacker can recover the plaintext simply by computing its integer $e$-th root.

Under the assumptions studied in the report, the attack becomes deterministic when:

$$
r \geq e.
$$

Thus, RSA can be compromised without knowledge of any private key when textbook RSA is used to broadcast the same plaintext with a common public exponent.

---

## 🔹 Part III — Integer Root Extraction

Since the classical attack ultimately requires computing:

$$
m=\sqrt[e]{X},
$$

the project compares two algorithms for extracting large integer roots:

- **Binary search**
- **Newton–Raphson iteration**

### Key Finding

Newton's method consistently outperforms binary search, particularly for larger exponents.

For exponents between approximately 20 and 100, the experiments report speed improvements on the order of **10× to 100×**. This is explained by Newton's much faster convergence, requiring approximately $O(\log\log X)$ iterations compared with $O(\log X)$ for binary search.

---

## 🔹 Part IV — Håstad Attack with Linear Padding

The study then considers a more sophisticated scenario where the plaintext is not directly encrypted, but transformed using a linear padding scheme:

$$
m_i' = a_i m+b_i.
$$

The resulting ciphertexts satisfy:

$$
c_i \equiv (a_i m+b_i)^e \pmod{N_i}.
$$

Using CRT, these congruences can be combined into a single polynomial equation:

$$
P(m)\equiv0\pmod{N_{\text{total}}}.
$$

The problem therefore becomes one of finding a **small root of a polynomial modulo a composite integer**.

---

## 🔹 Part V — Coppersmith & LLL

To solve the generalized attack, the project applies **Coppersmith's small-root technique**.

The method consists of:

1. Constructing a polynomial from the intercepted ciphertexts.
2. Building a lattice from polynomial multiples.
3. Applying **LLL lattice reduction** to find sufficiently short vectors.
4. Constructing a new polynomial whose small root is also the original secret message.
5. Recovering the message by solving the resulting integer polynomial.

The theoretical success condition is related to the size of the unknown message:

$$
|m| < N_{\text{total}}^{1/e-\epsilon}.
$$

---

## 📊 Main Experimental Findings

The experiments reveal an approximately linear relationship between the **critical number of recipients** and the RSA public exponent:

$$
\beta(e,k)\approx a(k)e+b(k).
$$

The experimentally obtained relationships were:

| `k` | Empirical threshold |
|---:|---:|
| 2 | $\beta_2 \approx 2.000e$ |
| 3 | $\beta_3 \approx 1.475e+0.450$ |
| 4 | $\beta_4 \approx 1.330e+0.341$ |
| 5 | $\beta_5 \approx 1.242e+0.382$ |

Increasing `k` reduces the number of recipients required for the generalized attack, but also significantly increases the computational cost of the **LLL lattice reduction**. This creates a trade-off between theoretical attack efficiency and practical computational complexity.

---

## 🛡️ Security Implications

The main conclusion of the study is that **RSA security does not rely solely on the difficulty of factoring large integers**.

The way plaintexts are padded before RSA encryption is equally important.

Deterministic or algebraically structured padding can preserve mathematical relationships between the plaintext and ciphertext, allowing attacks such as **Håstad + Coppersmith** to recover the original message.

In contrast, **RSA-OAEP** introduces randomness and cryptographic hashing into the padding process, making the encryption non-deterministic and breaking the simple polynomial relationship required by these attacks.

### Overall Conclusion

> **The project demonstrates how mathematical structure can turn RSA's algebraic properties into practical vulnerabilities, and why modern RSA encryption must use secure randomized padding such as OAEP.**