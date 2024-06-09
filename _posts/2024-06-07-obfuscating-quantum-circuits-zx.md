---
layout: post
title:  "Introducing the ZX Calculus"
author: Stefano Gogioso, Richie Yeung, Fabrizio Genovese
categories: ZX-calculus, quantum computing
excerpt: "TODO"
usemathjax: true
thanks: "TODO"
---

In our work towards practical Quantum 1-Shot Signatures, we have two main problems to solve:

- Building cryptographic gadgets called [equivocal hash functions](https://github.com/The-QSig-Commission/QSigCommissionWiki/wiki/Hash-function#equivocal-hash-function).
- Baking them into quantum circuits, and simplify the latter enough that the they can be run on quantum computers in the not-too-far future.

**TODO: Talk about why we need ZX**

<!-- In this post, we'll discuss an interesting aspect of our work which straddles the two problems above: part of the security of our equivocal hash functions might ultimately rely on obfuscation of the quantum circuits which implement them.

## Our Need for Obfuscation

Equivocal hash functions have been proved to exist by [Amos et al.](https://eprint.iacr.org/2020/107), using an abstract mathematical object known as 'ordered affine partitions'. Unfortunately, the proof of existence in their work is non-constructive, leading to the problem of concretely bulding such functions. We had a stab at concrete constructions (detailed in [this wiki page](https://github.com/The-QSig-Commission/QSigCommissionWiki/wiki/Hash-functions-from-ordered-affine-partitions) and demonstrated in [this Jupyter notebook](https://github.com/The-QSig-Commission/q1ss/blob/main/notebooks/AffinePartitions.ipynb)). Lev, one of the main researchers for this project, has identified a vulnerability in the current candidate implementation, but also proposed a fix which will be deployed in the near future.

Unfortunately, there is a catch. The above constructions &mdash; as well as other candidates we've been working on &mdash; need to be obfuscated for collision resistance to hold: if examined, our circuits readily yield an underlying secret which can be used to construct arbitrary collisions. As black box obfuscation is proven to be [infeasible](https://dash.harvard.edu/bitstream/handle/1/12644697/9034637.pdf), the current best bet for classical circuit obfuscation is [indistinguishability obfuscation](https://en.wikipedia.org/wiki/Indistinguishability_obfuscation) (here abbreviated 'iO', to avoid confusion with the more common meanings of the abbreviation 'IO').

iO has a lot of nice properties, but it is also presently very far from practical in the purely classical setting, and the resulting quantum circuits would be both punishingly expensive and painstakingly slow to run. While we are currently working on applications of iO &mdash; as well as other classical obfuscation techniques &mdash; to the circuits of interest in our research, we are also exploring more esoteric alternatives.

One of those esoteric alternatives is the subject of this post. We consider the following scenario:

- We have a family $(f_r)_{r \in R}$ of pseudorandom functions, and we instantiate a member of this family using some random secret $r \in R$.
- We bake the function into a quantum circuit $C$, but unfortunately the secret $r$ can be easily extracted from the structure of $C$.

We then ask: how can we rewrite the circuit $C$ into an 'obfuscated' circuit $K$ which computes the same function as $C$, but such that it is hard to extract $r$ from $K$? Enter, the ZX calculus. -->


<!-- ## The ZX Calculus -->

The [ZX calculus](https://en.wikipedia.org/wiki/ZX-calculus) is a diagrammatic language which can be used to design and reason about quantum circuits, originally devised by [Coecke and Duncan](https://arxiv.org/abs/0906.4725). A cool observation about it is that the very same rewrite rules which are used to reason about quantum circuits can also be leveraged to simplify them, performing the same computation while optimising some cost function of interest. For example, ZX-based techniques are State-of-the-Art for [T-count reduction](https://arxiv.org/abs/1903.10477) &mdash; useful in a future fault-tolerant quantum computation regime &mdash; and [2-qubit gate count reduction](https://arxiv.org/abs/2312.02793) &mdash; useful for both the future fault-tolerant regime and the noisy near-term regime.

The ZX calculus is limited to qubit-based digital quantum computing, but several extension exist which cover a broad spectrum of quantum computing paradigms. The [ZH calculus](https://arxiv.org/abs/1805.02175), for example, has notations and rewrite rules specialised for quantum computations making heavy use of binary logic gates, such as those appearing in our work. An extension of the ZH calculus from binary to [modular arithmetic](https://arxiv.org/abs/2307.10095) already exists, and an extension to arbitrary finite fields is upcoming; we expect both will be highly relevant to our work in the near future. Many other calculi exist, such as ones to reason about [photonic quantum computing](https://arxiv.org/abs/2306.02114),  or [continuous-variable quantum computing](https://arxiv.org/abs/2406.02905), and a searchable database of the literature can be found on the [ZX calculus website](https://zxcalculus.com/).

## Building Blocks

In the ZX calculus, quantum circuits are represented as networks of low-level building blocks known as 'spiders'. Spiders come in two 'colours', for the Z basis and the X basis respectively, and are shown below alongside the traditional [bra-ket notation](https://en.wikipedia.org/wiki/Bra%E2%80%93ket_notation) for the linear maps which they represent.

==Figure: generic Z (resp. X) spider with braket representation==

Spiders can be annotated by an angle, known as their 'phase'; when omitted, it is 0 by convention. A spiders can have any number $m \geq 0$ of input legs and any number $n \geq 0$ of output legs, and it represents a linear map from $m$ qubits to $n$ qubits. We now discuss some special cases of immediate interest in quantum computing. The 1-to-1 spiders are single-qubit rotations, about the Z and X axis, respectively:

==Figure: Z (resp. X) rotation==

The 0-to-1 spiders represent the single-qubit states &mdash; column vectors, or 'kets' &mdash; along the Z-axis equator of the Bloch sphere and its X-axis equator, respectively:

== Figure: Z (resp. X) spider states==

The 1-to-0 spiders are the 'adjoints' the states above &mdash; row vectors, or 'bras' &mdash;, and are used to represent the outcomes of certain single-qubit measurements:

== Figure: Z (resp. X) spider effects==

Even though the [Hadamard gate](https://en.wikipedia.org/wiki/Quantum_logic_gate#Hadamard_gate) can ultimately be defined in terms of spiders, as shown below, its usage in qubit-based quantum computing is so common that a special notation is available for it in the ZX calculus:

== Figure: the Euler definition of the H gate==

Spiders with $m$ input legs and $m$ output legs can be represented as $2^n$-by-$2^m$ complex matrices, composed in sequence by [matrix multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication) and in parallel by [Kronecker product](https://en.wikipedia.org/wiki/Kronecker_product). However, a more flexible semantics for spiders is as [tensors](https://en.wikipedia.org/wiki/Tensor) in a [tensor network](https://en.wikipedia.org/wiki/Tensor_network), where each connection between spiders corresponds to a [tensor contraction](https://en.wikipedia.org/wiki/Tensor_contraction). This shift of perspective is at the heart of recent breakthroughs in quantum simulation: for those interested in this topic, we recommend giving a look at the techniques by [Gray and Kourtis](https://arxiv.org/abs/2002.01935), implemented in the [cotengra](https://github.com/jcmgray/cotengra) library, or the work on the [Jet](https://github.com/XanaduAI/jet) library by [researchers at Xanadu](https://quantum-journal.org/papers/q-2022-05-09-709/).

We will henceforth adopt the interpretation of spiders as tensors, rather than complex matrices, to exploit the additional flexibility of graph-theoretic contraction semantics for tensor networks over the parallel-and-sequential composition semantics of complex matrices. We will use 'diagram' to indicate a network of spiders &mdash; a two-coloured graph, with optional angle annotations on nodes and optional Hadamard boxes on some edges &mdash; and reserve the 'tensor network' nomenclature to their concrete numerical interpretation (as one would use in [cotengra](https://github.com/jcmgray/cotengra) or [Jet](https://github.com/XanaduAI/jet)).

For the ZX calculus to be useful in digital quantum computing, it has to be able to represent all matrices/tensors between qubits: this property is known as 'universality' of the calculus.
To show universality, it is enough (cf. Section 4.5 of [Nielsen and Chuang](https://archive.org/details/QuantumComputationAndQuantumInformation10thAnniversaryEdition/)) to show that we can represent (i) arbitrary single-qubit rotations and (ii) the 2-qubit CNOT gate. Single-qubit rotations can always be obtained as a sequence of three alternating Z-X-Z rotations, for suitable choice of angles: we already introduced these above, as 1-to-1 spiders. For the CNOT gate, on the other hand, we need two more instances of spiders: the 1-to-2 copy spider and the 2-to-1 XOR spider, shown below.

==Figure: the copy and XOR spiders==

Combined, the copy and XOR spiders form the CNOT gate, the most common 2-qubit gate used in digital quantum computing:

==Figure: the CNOT gate==


### Rewrite Rules

Above, we have shown that the ZX calculus is 'universal': it can represent all tensors between qubits, and hence all computations in digital quantum computing. This makes it useful to write down quantum circuits, but not yet to reason about them. To gain reasoning powers, we need to introduce 'rewrite rules': rules to rewrite small ...

**TODO: Got here with writing**

<!-- ## Quantum Circuit Obfuscation with the ZX Calculus

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

Hopefully, this will be material for a future blog post, so stay tuned! -->