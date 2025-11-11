
***
# Privacy Amplification

As previously, we have our protagonists Alice and Bob, trying to communicate in a secure manner, and an *eavesdropper* Eve, trying to intercept the communication.

As we have seen in **Session 1**, Alice and Bob need a *shared private key* to be able to communicate.

If Alice and Bob know each other well, they might attempt to build a private key from common shared knowledge, such as the flavor of the first cone of ice cream they shared, or the name of Bob's cats. Doing so will yield a somewhat "fairly" secret key.

> [!info] 
> This key will only be fairly secret, as other individuals might also know such shared information, and Eve might be able to gather some through various means.

Naturally, Alice and Bob want to find a scheme to create an actually secret key from this fairly secret starting key.

Introducing privacy amplification :

Alice and Bob share a "weak secret" $x$ which comes from some distribution $p_x$ , represented as a random variable $X$ called the *source*.
While the distribution of $X$ may not be known, $x$ is available to both parties.

Eve possesses side information $E$ which might be correlated to $X$, such as its first bit, its parity, or even some quantum state $\rho_x^E$.

The goal for Alice and Bob is to generate some $z$ such that from the point of view of Eve, $z$ (chosen from a distribution $Z$) is (close to) uniformally random.

$$
\rho_{XE} \mapsto \rho_{ZE} \approx_\epsilon \frac{\mathbb{I}_Z}{|Z|}\otimes \rho_E
$$
> [!tip] Note
> The size of $z$ will depend on how secret the original secret $x$ is. If $X=E$ there is not much that can be done.
> Intuitively, this will be quantified by the min-entropy $H_{min}(X|E) = -\log P_{guess}(X|E)$.
# Randomness Extraction

Let's first start with a simpler problem, Alice has some source $X$ from which she gathers a string $x$, which is possibly correlated with $E$, with the promise that $H_{min}(X|E) \geq k$ (with this condition, $X$ is called a $k$-source), and is tasked to generate $z$ such that the distribution $Z$ is close to uniform even conditionned on $E$. Similar setup as before, but Alice is on her own and does not have to coordinate with bob.

> [!tip] Note
> While Alice could just sample from an actually uniform distribution, or measure $\ket{0}$ in the Hadamard basis a bunch of times, assume here that randomness is *costly*, and Alice does not have ready access to large amounts of randomness for free. The goal is to ***extract*** randomness from X.
## A simple example

If $X$ has a distribution consisting of $n$ i.i.d. variables $p_i$, one can extract a single bit *nearly* uniformly by taking the parity of all the bits

> [!Example]-
> $X = (X_1, ..., X_n)$ i.i.d., take $Z = X_1 \oplus X_2 \oplus ... \oplus X_n$.
> If $X_i$ has probability 3/4 of being 1, 1/4 of being 0, for $n=2$ we have $P(Z=0) = 0.625$, $P(Z=1) = 0.375$. Not quite uniform, but better than what we started with, and better still for increasing $n$.

One can use the same approach for variables which aren't identically distributed, and get a single bit $Z$ close to a uniform distribution. However this is not very efficient.
## Seeded extractors

Using a deterministic function to extract randomness (such as $Ext(X) = X_1 \oplus ... \oplus X_n$) can only get you so far however, since for any $k$, even if $H_{min}(X|E) \geq k$, there is no function that can *always* extract $k$ bits of randomness from $X$.

Instead, we need to expand the scope to *seeded* extractor functions, which use a *seed* as a source of randomness. However, since this randomness may be costly, the seed should be of relatively small size compared to $X$ and $k$.

> [!definition]
> A $(k,\epsilon)$ *weak seeded extractor* is a function Ext : $\{0,1\}^n \times \{0,1\}^d \mapsto \{0,1\}^m$ such that for any $k$-source $\rho_{XE}$,
> $$D(\rho_{Ext(X,Y)E}, \frac{\mathbb{I}}{2^m}\otimes \rho_{E}) \leq \epsilon$$
> With $Y$ uniformally distributed, independent from $X$ and $E$.

One might think that one could simply use the seed as randomness, but Alice and Bob could not choose the same seed as a private key without communicating it publicly.

This induces the definition for a strong seeded extractor, where the extractor's input is independent from the seed

> [!definition]
> A $(k,\epsilon)$ *strong seeded extractor* is a function Ext : $\{0,1\}^n \times \{0,1\}^d \mapsto \{0,1\}^m$ such that for any $k$-source $\rho_{XE}$,
> $$D(\rho_{Ext(X,Y)YE}, \frac{\mathbb{I}}{2^m}\otimes \rho_{YE}) \leq \epsilon$$
> With $Y$ uniformally distributed, independent from $X$ and $E$.

## Why the min-entropy?

Intuitively, the min-entropy gives an estimate of how much randomness there is to extract from $X$, but it is a valid question to seek if a strong seeded extractor really can extract $k$ bits from $X$ if it has $H_{min}(X|E) \geq k$ (meaning it is a $k$-source).

If $H_{min}(X|E) \geq k$, it is impossible to extract more than $k$ bits from $X$ while still being uncorrelated to $E$, because you can guess the value of the extractor function by guessing $X$ first then applying the function, so $\forall f, H_{min}(f(X)|E) \leq H_{min}(X|E)$, in particular $Ext(., Y)$.

Conversely, we will show strong seeded extractors which achieve the extraction of a (mostly) uniform $k$ bits given $X$ being a $k$-source.
# Hashing

To build a strong seeded extractor, we will make use of hashing functions, which essentially take a long string and map them to shorter strings somewhat uniformally.

We need to introduce classes of hashing functions families, namely 1-universal and 2-universal hash function families :

> [!definition] 1-universal family
> A family $\mathcal{F}$ of hashing functions $\{0,1\}^n \mapsto \{0,1\}^m$, $m\leq n$ is 1-universal if for every $x \in \{0,1\}^n$ and $z \in \{0,1\}^m$,
> $$P_{f \in \mathcal{F}}(f(x)=z) = \frac{1}{2^m}$$

Notice that the definition has $x$ and $z$ fixed, and the probability is over the family of functions.

While a 1-universal family is enough to build a *weak* extractor, the following definition will let us build a *strong* one :

> [!definition] 2-universal family
> A family $\mathcal{F}$ of hashing functions $\{0,1\}^n \mapsto \{0,1\}^m$, $m\leq n$ is 1-universal if for every $x,x' \in \{0,1\}^n, x \neq x'$ and $z,z' \in \{0,1\}^m$,
> $$P_{f \in \mathcal{F}}(f(x)=z \land f(x')=z') = \frac{1}{2^{2^m}}$$

Once given such a family, the extractor will use the seed to choose a function in the family, and apply it to $x$ to obtain $z$. While the set of all functions is a 2-universal family, it has too many functions, and the seed would need to be too large to be practical.

Instead, we will take functions in $\mathbb{F}_q$ the field with $q=2^n$ elements, of the form $$a,b \in \mathbb{F}_q, f_{a,b}(x) = ax+b$$
where multiplication and addition are done in $\mathbb{F}_q$. This yields a family of $2^{2n}$ functions which so happens to be a 2-universal family of hashing functions.

> [!faq]- Proof
> Let $x \neq x' \in \mathbb{F}_q, (z,z') \in \mathbb{F}_q$, the probability of $f_{a,b}(x)=z \land f_{a,b}(x')=z'$ is the probability of $a(x'-x)=z'-z$, so $a=\frac{z'-z}{x'-x}$ and $b=z-ax$. This gives us a single possible choice for $a$ and $b$, among $2^{2m}$, giving us the correct probability.

Notice how the functions map $n$ bit strings to $n$ bit strings, but we can simply toss the extra $n-m$ bits we don't want such that we are given $m$ bit strings, which still retains the 2-universality property.

Our extractor will now be defined as $$ Ext_{\mathcal{F}}(x,y) = f_y(x)$$
# How good is Hashing?

To analyze the quality of the hashing functions, we will make use of the *leftover hash lemma*, which has different formulations depending on whether we consider quantum side information or not.
## No side information case

In this case, we don't take into account $E$ from Eve, in which case the lemma is stated as follows:

> [!info] Lemma
> Let $k\leq n$ integers, $\epsilon > 0$, $m = k - 2 \log(1/\epsilon)$, and $\mathcal{F}:\{0,1\}^n \to \{0,1\}^m$ a 2-universal family of hash functions. Then $Ext_{\mathcal{F}}$ is a $(k, \epsilon)$ strong seeded extractor.

With our construction, the seed needs to be of length $d=2n$ to be able to choose the hashing function, which is longer than the optimal seed length for the general case, but in the case of privacy amplification it is satisfactory. In the case without side information, we have $H_{min}(X) \leq k$.

To prove this lemma, we will proceed in two steps. First we want to show that the error can be bounded by first showing a bound on the *collision probability*, and then we bound this *collision probability* and achieve the result of the bound on the error.

#### From error to collision probability

As mentionned, we want to bound the error $D(\rho_{Ext(X,Y)Y}, 2^{-(m+d)}\mathbb{I})$ by something related to the collision probability $\sum_{z,y}p^2_{zy}$ where $z = f_y(x)$, and $$p_{zy} = Pr[(Ext(X,Y), Y) = (z,y)] = 2^{-d}\sum_{x/f_y(x)=z} p_x.$$
> [!faq]- First part of the proof
> $$ D(\rho_{Ext(X,Y)Y}, 2^{-(m+d)}\mathbb{I}) = \frac{1}{2}\sum_{z,y} \left| 2^{-d}\sum_{x/f_y(x)=z}p_x - 2^{-(m+d)} \right| $$ Using Cauchy-Schwartz inequality,
> $$ \leq 2^{\frac{m}{2} -1}\left(2^{-d}\sum_{z,y} \left|\sum_{x/f_y(x)=z}p_x - 2^{-m}\right|^2\right)^{1/2} $$ After expanding the square and doing much simplification,
> $$ = 2^{\frac{m}{2} -1}\left(2^d \sum_{z,y} p_{zy}^2 - 2^{-m} \right)^{1/2}.$$

#### Bounding the collision probability

Here we make use of the 2-universal family property to bound the collision probability $\sum_{z,y}p^2_{zy}$.

> [!faq]- Second part of the proof
> By expanding the square in the definition, we have
> $$ \sum_{z,y}p^2_{zy} = 2^{-2d}\sum_{y,z}\sum_{x,x' / f_y(x)=f_y(x')}p_xp_{x'}$$
> $$ = 2^{-2d}\sum_{z,y}\left(\sum_{x \neq x'}p_xp_{x'} + \sum_xp_x^2\right)$$
> Using 2-universality,
> $$ = 2^{-(d+m)}\sum_{x \neq x'}p_xp_{x'} + 2^{-d}\sum_xp_x^2$$
> Using the property of the minimal entropy being bounded by $k$,
> $$\leq 2^{-(d+m)} + 2^{-(d+k)}$$
> Plugging this back into the first part we get that the error is bounded by
> $$2^{\frac{m-k}{2}-1}$$ proving the lemma.

## Side information case

To model side information, we will take $\rho_{XE}$ to be a cq-state $\sum_x \ket{x}\bra{x} \otimes \rho_x^E$ such that $H_{min}(X|E) \geq k$.

Before proving the statement with side information, we need to dive into the measurements Eve can make. Indeed, finding the optimal measurement to guess $X$ can be solved via semi-definite programming, but there is no formula whenever $|X| > 2$. Instead, we will have to consider *pretty good* measurements.

#### Pretty good measurements

Choosing the measurement $M_x = \rho_x^E$ is positive semidefinite, but not necessarily normalized such that $\sum_x M_x = \rho_E$, except when the $\rho_x^E$ are perfectly distinguishable. We can instead define $$M_x = \rho^{-1/2}\rho_x\rho^{-1/2}$$ where $\rho = \sum_x\rho_x$ and we use the Moore-Penrose pseudo-inverse, and $M_x$ is a POVM. One can show that the probability of guessing $X$ using $E$ and this POVM is the square of the optimal probability given any POVM.

The proof with this pretty good measurement of the quality of our strong seeded extractor follows the same structure as the previous one, however it is quite more involved. If you do want to check it out, it starts page 13 of [Lecture 4](https://ocw.tudelft.nl/wp-content/uploads/LN_Week4.pdf).
# Back to cryptography

Using the results we have just shown, we can now devise a relatively simple protocol for privacy amplification:

> [!tip] Privacy Amplification Protocol
> 1. Alice and Bob build their shared *weak* secret $X$, which might be correlated to Eve's $E$.
> 2. Alice chooses a random seed $Y$ for the extractor, and sends it to Bob through the public channel.
> 3. Alice and Bob each compute $Z = Ext(X,Y)$ using the extractor built with hashing functions over the finite field with $2^{|X|}$ elements.

This protocol is always correct since Alice and Bob perform the same computation (assuming they have the correct shared $X$ to start with).

It is also secure, as our seeded extractor gives us the guarantee that $Z$ is close to uniform, even conditionned on $E$ **and** $Y$. By using the 2-universal family of hash functions to build the strong seeded extractor, the protocol is now complete.

This will be useful for next session, which will discuss key distribution protocols.