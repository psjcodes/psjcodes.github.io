---
title: Thread - The Basis Is the Bottleneck
created: 2026-06-22
modified: 2026-06-23
---

## Background
I've spent a while now thinking about one question: *can we read the structure of a trained network back out of it?* Looking back on my prior work on recovering Boolean functions and my ongoing explorations into grokking phase transitions, SAE dilution, and recovering compositional structure, I seem to arrive at the same answer every time. 

The structure is real, and networks do often acquire it; what fails is the *readout*, and a better optimizer or more capacity doesn't save us. But "the readout fails" hides two different walls, and keeping them apart is the whole point. The first is **identifiability**: the observations do not determine the latent structure on their own, so no estimator recovers it until we supply side information, a symmetry group, a sparsity prior, a hypothesis class, or a privileged coordinate system, that breaks the underdetermination. The second is **perturbation geometry**: even once the structure is identified and behaviorally recovered, expressing it through a change of basis that is poorly conditioned (not norm-preserving) amplifies a small behavioral error into a large change in the coordinates, so any criterion that demands exact equality of representations rejects a solution that is in fact behaviorally correct.

The bottleneck is rarely *finding the true latent basis*; more often it is *choosing a representation whose perturbation geometry matches the recovery claim we want to make*. "The basis is the bottleneck" is the slogan, but "basis" is doing two jobs, the side information that resolves identifiability and the measurement representation that keeps a recovery claim stable, and most of the confusion comes from conflating the two.

## Definitions
A few of these words get overloaded, so to be precise:

- **Representation (basis):** a coordinate system in which we express a learned function. Bases related by a change of basis carry the same information but differ in *perturbation geometry*, how a small change in behavior maps to a change in coordinates: an isometric (norm-preserving) change of basis keeps it proportional, a poorly conditioned one amplifies it.
- **Recovery:** the claim that the network's function agrees with a target hypothesis. It comes in three strengths: *behavioral* (the functions agree except on a small fraction of inputs), *decoding* (within a restricted hypothesis class, the target is the unique nearest admissible hypothesis), and *symbolic* (exact equality of the representations). These coincide only at zero error.
- **Certificate:** a verifier that confirms recovery against a supplied candidate, rather than discovering the target from scratch, typically by testing membership in a uniqueness region around the candidate.
- **Side information:** the structural prior that must be supplied to turn fit or reconstruction into recovery, e.g. a symmetry group, a sparsity prior, a hypothesis class, or a privileged coordinate system. It is what resolves the identifiability underdetermination.
- **Gauge:** a transformation the observations cannot distinguish (e.g. a relabeling of latent coordinates). Recovery is defined only up to this equivalence class; the gauge-invariant quantities are the ones that can be identified.

## Subthread - Recovering learned Boolean functions
The cleanest demonstration of "the basis is the bottleneck" is a result I can state exactly; the results from [my workshop paper](https://openreview.net/forum?id=rOuUpMXyvH). We train an MLP on parity at n=16; it classifies 99.3% of inputs correctly, and so behaviorally we conclude it has learned parity. Since the input-output pairs are fully enumerable, we can read out the learned function in its algebraic normal form (the monomial basis, via Mobius transform) and we get a degree-16 polynomial with ~25,000 monomials, whereas true parity is degree-1 with support 16. The network recovered the rule in every behavioral sense, but the exact symbolic readout blows up due to a few hundred residual bit-errors pushed through the Mobius transform's non-locality. I call this regime, where the behavior has recovered the rule but the symbolic readout has not, the **Mobius noise zone**.

The fix isn't a better network, but a better basis to read out in. The three notions of recovery come apart in exactly this setting: the network agrees with parity on 99.3% of inputs (**behavioral**), and among low-degree rules it is uniquely nearest to parity (**decoding**), yet **symbolic** recovery, exact equality of the representations, fails by orders of magnitude. The behavior is intact; the exact symbolic readout is the brittle thing.

To separate these, we use two classical tools that stay stable where symbolic equality is brittle:

- **Parseval's theorem in the Walsh-Hadamard basis:** the Walsh-Hadamard spectrum is the function rewritten in the parity basis, and that change of basis is an orthonormal rotation, so it preserves distances. Parseval makes it exact: the squared distance between the network's and the rule's spectra equals 4δ, four times the fraction of inputs they disagree on. So the spectral distance isn't a proxy for the bit-error rate, it is the bit-error rate, read as one continuous number. And because a rotation can't blow up a small perturbation, the measurement degrades linearly with error, where the Mobius/ANF transform sprays one bit-flip across many coefficients and explodes.
- **Reed-Muller decoding radius:** rules of a given degree sit far apart (any two differ on at least 2^(n−r) inputs) so a ball of half that radius around the target holds no other degree-r rule. If the network's truth table lands inside it, the target is provably the unique nearest degree-r rule: the behavior decodes to the rule, and a few hundred stray bit-errors don't flip the verdict because the ball has room to spare. A binary certificate, not a measurement; it needs only the rule's degree, which sets the radius; simpler rules give bigger balls and absorb more error.

So even in the most transparent setting, the representation we read out in decides whether we see the structure or a mess. Note what kind of failure this is: not basis *discovery* (we have the right basis), but the *instability of an exact symbolic readout* under small error. The fix is to read out in a representation whose perturbation geometry matches the claim, a rotation like Walsh that keeps a small error small, rather than the Mobius basis that amplifies it.

## Aside - Isn't this obvious? We can't identify the natural basis
The anticipated objection: *of course we can't read out canonical structure without supplying a basis, that's just unsupervised representation learning being underdetermined*. Yes, it's impossible. But the impossibility isn't the binding constraint, and that's the part I think isn't obvious.

The Mobius noise zone has nothing to do with failing to find the basis: there we have the canonical basis, the rule is behaviorally recovered, and yet the readout still blows up. That brittleness is a different wall altogether, and is the one that bites first. Lumping it under the "can't identify the basis" category is a misdiagnosis.

Accepting the impossibility buys us things rather than ending the conversation. We can certify behavioral recovery against a supplied candidate without identifying anything (a question that practitioners actually have). The indeterminacy is a gauge quotient, so the invariant we can recover is exactly the one we needed; "non-identifiable" is not the same as "nothing survives." The residual is a structured signal we can act on. Whether we're at the statistical or computational wall is a measurement, not the end of the world.

Also, if "we can't identify the natural basis" were really treated as obvious, the field wouldn't be pushing for finding canonical SAE atoms and circuits, which are deliverables that quietly assume that basis-free identification is achievable. So either it isn't operationally obvious, or what follows from it (don't chase canonical recovery, certify behavioral recovery and read the residual instead) should already be standard, but it isn't.

The contribution isn't the impossibility, it's that taking it seriously reorganizes the problem into more tractable questions, and a lot of good work is still solving the intractable version of it. Some of these moves aren't new on their own (certifying instead of discovering, the gauge quotient, both live in adjacent literatures); the claim is narrower, that the same structure recurs across all four subthreads, and that the operative failure is usually brittleness, not impossibility.

## Subthread - Grokking modular addition
If we want to study recovery, then we want a setting where we know the structure exists. Nanda et al.'s[2] grokking transformer gives us this for free.

On modular addition, the network learns to place inputs on a circle and compute in the Fourier basis (Nanda et al., 2023). We can watch that happen with a simple concentration measure on the embedding's Fourier power spectrum, the *participation ratio*: roughly the effective number of frequencies the energy spreads across, high when it's smeared over many modes and low when it piles onto a few. Across the grokking transition that ratio collapses by roughly 6.6x, and does so as a leading ramp, before the sudden behavioral jump, so the clean algebraic object (the circle) becomes visible in the weights *when we read it in the privileged basis* (can read further on this [[Notes/grok-fourier-pr.md|here]]). This matters because, when recovery fails on harder tasks, we can't just say "the structure isn't there"; in this case it's cleanly measurable.

So this isn't just a story about late generalization, but also that grokking is a control condition for interpretability. In this case where the latent structure is known, the transition is visible, and the recovery question can be answered properly. Roughly, we can write this:

`z_t(x) = z_rule,t(x) + r_t(x)`

Early on, the residual/memorization component carries the train behavior, and later, the rule component becomes stable and test accuracy snaps. In the Boolean case, we can examine this residual component directly: `r = f XOR h`, where f is the learned function and h is the target. The behavioral transition is a thresholded view of the underlying representational change. Here, grokking is about the dynamics of how a system moves from a long, high-complexity code to a short, structured one, with respect to the privileged basis.

## Subthread - Hopes on SAEs might be too high
*Stubs:*

- **SAEs work, and the authors' own caveats are the thesis:** Scaling monosemanticity[3] pulls millions of interpretable, even safety-relevant, features off a frontier model; the same work flags non-uniqueness across training runs, unknown completeness, and that the result is "partly illusory." Those caveats are the identifiability wall, stated from the inside.
- **The failure modes sort into the two walls:** The SAE-eval literature[4] reads as a grab-bag (absorption, feature splitting, non-uniqueness, dead features, interpretability illusions), but it isn't: non-uniqueness / absorption / splitting are *identifiability* (which decomposition); shrinkage, and the SAEBench finding that better reconstruction doesn't improve disentanglement, are *perturbation geometry*; recall-without-precision illusions are a *certificate* failure (no verification against a candidate). Gated / TopK / JumpReLU fix the measurement and still don't cross identifiability, which is the whole point.
- **Non-uniqueness isn't a surprise, it's overcomplete ICA:** "Independently trained SAEs learn different dictionaries" is the non-identifiability of an overcomplete code without side information[5], the gauge from the two-wall framing above, not a new empirical wrinkle.
- **Principled vs incidental privileged basis:** Grokking gets its basis from a symmetry (Fourier); the residual stream gets one from Adam's per-dimension normalizers[6], an *incidental* basis rather than a principled one. An incidental basis beats random but doesn't buy identifiability, which is exactly why residual- or SAE-basis features are "one valid decomposition," not canonical; the gauge is free in theory but partially pinned by training dynamics.
- **Useful is not canonical:** You can steer effectively in SAE latent space[7] without the features being the model's true units; that's the pragmatic frontier[8], and it's fine, as long as we don't sell the handle as the structure.

## Subthread - Reference-free recovery up to gauge
*Coming soon*

## Conclusions
1. **The failure is in the readout:** Sometimes the readout basis is brittle (the Mobius noise zone: a perfectly good solution detonates in the wrong basis), sometimes the readout is statistically underdetermined (SAE dilution at a Fano boundary; the gauge orbit). Both are cases of "the basis is the bottleneck," and both tell us where to spend effort: choosing a stable basis, supplying structure, or accepting recovery-up-to-symmetry.
2. **Reconstruction is not recovery, and we often conflate them:** SAEs are a good reconstruction tool, but the geometric wall and dilution results prevent it from becoming a certificate of identifiability, and the field has discovered this gap. Identification needs a different, information-supplying step on top.
3. **Knowing when to abstain is a good thing:** A method that recovers structure *and tells us when it can't* is worth more than one with a higher headline number and no idea where its blind spots are.

None of this gets us canonical, basis-free, unsupervised interpretability. The right basis is either brittle to read out exactly or underdetermined without side information, a fact at near-theorem status, and no amount of engineering can help us dig it out. The honest frontier could very well be finding methods that read out in a stable basis, reach the identifiability boundary, and are candid about living up to gauge, which is a smaller promise than "we'll reverse engineer the network," but it's probably one we can safely keep.

## References
1. Pranay Jha. *Noise-Tolerant Verification of Compositional Boolean Recovery.* 2nd Workshop on Compositional Learning, ICML 2026. [OpenReview](https://openreview.net/forum?id=rOuUpMXyvH)
2. Neel Nanda, Lawrence Chan, Tom Lieberum, Jess Smith, Jacob Steinhardt. *Progress Measures for Grokking via Mechanistic Interpretability.* ICLR 2023. [arXiv:2301.05217](https://arxiv.org/abs/2301.05217)
3. Templeton et al. *Scaling Monosemanticity: Extracting Interpretable Features from Claude 3 Sonnet.* Transformer Circuits, 2024. [summary](https://learnmechinterp.com/topics/scaling-monosemanticity/)
4. *Sparse Autoencoder Variants and Evaluation* (incl. SAEBench). learnmechinterp.com. [link](https://learnmechinterp.com/topics/sae-variants-and-evaluation/)
5. Aapo Hyvärinen, Petteri Pajunen. *Nonlinear Independent Component Analysis: Existence and Uniqueness Results.* Neural Networks, 1999.
6. Nelson Elhage et al. *Privileged Bases in the Transformer Residual Stream.* Transformer Circuits, 2023. [link](https://transformer-circuits.pub/2023/privileged-basis/index.html)
7. Samuel Soo, Chen Guang, Chandrasekaran Balaganesh, Wesley Teng, Tan Guoxian, Yan Ming. *Interpretable Steering of Large Language Models with Feature Guided Activation Additions.* arXiv:2501.09929, 2025. [arXiv:2501.09929](https://arxiv.org/abs/2501.09929)
8. Neel Nanda, Josh Engels, Arthur Conmy, et al. *A Pragmatic Vision for Interpretability.* AlignmentForum, 2025. [link](https://www.alignmentforum.org/posts/StENzDcD3kpfGJssR/a-pragmatic-vision-for-interpretability)

*(More to come as the SAE and reference-free subthreads are written.)*

### AI Disclaimer
*A large language model (LLM) was used in the drafting and editing of this content. The author reviewed and refined the text to ensure accuracy and maintain personal voice.*



