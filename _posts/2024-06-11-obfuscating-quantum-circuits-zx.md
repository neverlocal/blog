---
layout: post
title:  "ZX-based quantum obfuscation for dummies"
author: Fabrizio Genovese
categories: ZX-calculus obfuscation
excerpt: This is all conjectures for now, but nice conjectures nevertheless!
usemathjax: true
thanks: I want to thank Stefano Gogioso and Richie Yeung, which are actively working to figure out this stuff.
asset_path: /assetsPosts/2024-06-12-obfuscating-quantum-circuits-zx
---

The quest around building Quantum 1-Shot Signatures revolves around building practical, optimized [equivocal hash functions](https://github.com/The-QSig-Commission/QSigCommissionWiki/wiki/Hash-function#equivocal-hash-function).

Such equivocal hash functions have been proved to exist, for instance in [this paper](https://eprint.iacr.org/2020/107), using ordered affine partitions. Unfortunately, the proof there is essentially non-constructive, so the problem of 'actually building the thing' still stands. We had a first stab at building these gadgets explicitly [here](https://github.com/The-QSig-Commission/QSigCommissionWiki/wiki/Hash-functions-from-ordered-affine-partitions), but Lev, one of the main researchers involved in this project, quickly broke the mechanism.

Now we have some other candidate constructions: Lev is working on implementing Quantum 1-shot tokens using claw-free functions -- hopefully a post about this willfollow soon! -- whereas Stefano and Richie are investingating an alternative approach based on [ZX-calculus](), which will be the focus of this post.

In general, all of these endeavours piggyback to the same problem, namely that the construction used to implement an equivocal hash function needs to be *obfuscated* in some way. This has a very precise meaning in cryptography, but the layman interpretation of 'obfuscated' would be 'turn the code into a sort of black box: people can feed inputs and read outputs, but have no clue about how the program behaves'. This is called 'black-box obfuscation', and has been unfortunately proven to be [infeasible](https://dash.harvard.edu/bitstream/handle/1/12644697/9034637.pdf). The next best thing is called [indistinguishability obfuscation](https://en.wikipedia.org/wiki/Indistinguishability_obfuscation) -- abbreviated 'iO' -- and, citing Wikipedia, it means more or less something like this:

> In cryptography, indistinguishability obfuscation (abbreviated IO or iO) is a type of software obfuscation with the defining property that obfuscating any two programs that compute the same mathematical function results in programs that cannot be distinguished from each other. Informally, such obfuscation hides the implementation of a program while still allowing users to run it.

This seems rather harmless but in practice allows us to implement a ton of very well-known crypto primitives, such as [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography), [deniable encryption](https://en.wikipedia.org/wiki/Deniable_encryption) and [functional encryption](https://en.wikipedia.org/wiki/Functional_encryption). So, Io has a lot of nice properties, and assuming it gives us a very nice tool for building crypto primitives in a theoretical context. Unfortunately, it is totally impractical at the moment, so we cannot really use it to build production stuff.

So, whereas we play with indistinguishability obfuscation and other gadgets on one hand, we are also exploring more esotheric alternatives on the other hand. This basically translates into 'obtaining something like iO in our particular case by cutting as many corners as possible'. One of these esotheric alternatives is what this post is about. To make this explanation more palatable to the layman, from now on we'll fix the following scenario:

> Suppose that I have a pseudorandom function, which I instantiate with some random seed $r$. Suppose furthermore that I bake this thing into a (quantum) circuit. How can I rewrite the circuit so that it is difficult to 'extract' $r$ by analyzing it?

You see from this how it is not unreasonable for us to believe that we can cut some corners from the full iO setting: After all, a pseudorandom function is difficult to invert if one doesn't know the corresponding quantum seed.


Essentially, we need to bake our function, instantiated with a random seed $r$, into a quantum cicuit. This is nothing more than a bunch of (quantum) gates, which will be passed to a quantum computer to be run. Needless to say, if someone can read $r$ the whole protocol becomes insecure, so the problem is: how do we make sure people won't be able to extract $r$ by just looking at the gates of the circuit implementation?

## Enter the ZX calculus

The [ZX calculus](https://en.wikipedia.org/wiki/ZX-calculus) is a diagrammatic calculus to express quantum circuits. The cool thing about is is that it has a bunch of very disciplined rewriting rules which can be leveraged to simplify a quantum circuit. For instance, you can take a quantum circuit, translate it into a ZX diagram, and apply the rewriting rules until you obtain a circuit that:

1. Computes the same thing;
2. Has some nice properties, for instance it may use a lower [Toffoli gate](https://en.wikipedia.org/wiki/Toffoli_gate) count.

The ZX calculus looks like the picture below. In the top half of the picture, you can see the ZX calculus 'atoms', notated with the unitaries they correspond to in 'standard' quantum computing. We call these 'spiders'. The red and green spiders on rows 3 and 4 can be obtained as a special case of the $(n,m)$-legged spiders in rows 1 and 2 (they are just $(1,1)$ and $(0,1)$ spiders, respectively), and are listed explicitly only for clarity. There is also the Hadamard gate, not defined explicitly because it's really like its traditional counterpart, which is represented by a yellow box marked 'H'.

![ZX calculus recap table]({{page.asset_path}}/zx-calculus.png)

Spiders are connected by merging their legs, and in the bottom half of the picture you can see the rewriting rules. So, for instance, rule (C) says that you can always turn a green spider into a red one by applying a Hadamard gate to each of its legs. A [very deep result](https://arxiv.org/pdf/1706.09877) says that each unitary in ${\mathbb{C}^{2}}^{n}$ can be turned into a ZX diagram, and this in turn means that every quantum circuit can be turned into a bunch of spiders.

## Obfuscating circuits

The underlying idea of this particular obfuscation technique we are pursuing is the following: Take for instance rules (S2): These tell you that every time you have a wire, you can introduce some spiders on it. So we could take a circuit, introduce some *redundant* spiders using (S2) and then *let them diffuse* through the circuit applying the other rules. In a nurshell, this amounts to introduce 'harmless noise' - that is, noise that cancels out completely, but then letting diffuse it into the circuit in a way so that it becomes impossible to distinguish the noise from the original gates.

In a nushell, by diluting the circuit with 'noise' we let the information about the random seed $r$ of our running example dissolve into it, in a way that is not easily recoverable.

In practice, this method relies on something called [Pauli gadgets](https://arxiv.org/pdf/1906.01734), which are exactly the lego bricks we need to add to our circuits to produce the noise we need and then diffuse it. To do this properly though, we need to figure out some stuff about how these gadgets commute between each other under particular conditions we are interested in. We are exactly doing so on one hand, wereas we prepare code mocks on the other hand.

### Linear algebraic explanation

If you are more versed with linear algebra, see it this way: Immagine I have a composition of unitaries:

$$ U_1 U_2 U_3 \dots U_n.$$

Going back to our running example, one or more of these unitaries will encode my random seed $r$ in some way. If I want to make this information hard to extract, I can rewrite the composition above as:

$$ U_1 I U_2 I U_3 I \dots I U_n.$$

Basically, I have introduced identity matrices $I$ everywhere, which by definition do not alter the computation. Now, I can rewrite each $I$ as $V^{-1}V$, where $V$ is some unitary matrix. If I pick the $V$s wisely, so that they commute with the $U_i$ in some non-obvious way, I can end up having a circuit 

$$ W_1 W_2 W_3 \dots W_m.$$

Where the $W_j$ and the $U_i$ really don't look like anything alike. The idea here is that there are really *many alternatives* - as in, a continuum of them - for the $V$s, so that reconstructing the original circuit may not be computationally easy.

The main reason to prefer the ZX approach to the algebraic one is that the 'diffusion step' in the ZX case is merely graph rewriting, which has already been efficiently implemented in a plethora of ways.

Again, this is a bit of an oversymplification. In reality, Pauli gadgets are not exactly just 'unitaries', but the infinitesimal generators of the [Lie algebra](https://en.wikipedia.org/wiki/Special_unitary_group#Lie_algebra) $\mathfrak{su}(n)$ associated to the [Special unitary Lie group](https://en.wikipedia.org/wiki/Special_unitary_group) $SU(n)$. In the case of $\mathfrak{su}(2)$, Pauli gadgets are some of the most important single-qubit operations. Part of our work is figuring out the commutation relations between 'higher Pauli gadgets' - that is, generators of $\mathfrak{su}(n)$ for $n \geq 3$ - in purely diagrammatic terms. This would allow us to use the already developed diagrammatic-rewriting software for the ZX calculus to try this sort of obfuscation techniques.

## Will this work?

God knows! Obviously coming up with such an idea and proving that it is computationally secure are two very different beasts. We have some proof ideas that involve random walks on a graph and some other stuff, but it's still too early to say. Most importantly, we need to get more clarity around which complexity assumptions we would have to make so that such a proof could hold.

Hopefully, this will be material for a future blog post, so stay tuned!