---
title: Thread - The Basis Is the Bottleneck
created: 2026-06-22
modified: 2026-06-23
---

## Background
I’ve spent a while now thinking about one question: *can we read the structure of a trained network back out of it?* Looking back on my prior work on recovering Boolean functions and my ongoing explorations into grokking phase transitions, SAE dilution, and recovering compositional structure, I seem to arrive at the same answer every time. 

The structure is real, and networks do often acquire it; the hard part is specifying the basis we read it out in, and when that doesn't work, the constraint is a brittle representation or an identifiability wall, and a better optimizer or bigger SAE doesn't save us. The observations don't specify the latent structure until we supply it the right side information. That side information can be symmetry, a sparse dictionary, or a task basis; whatever form it takes, it does the same job of turning fit or reconstruction into recovery.

## Subthread - Recovering learned Boolean functions
The cleanest demonstration of “the basis is the bottleneck” is a result I can state exactly; the results from my workshop paper. We train an MLP on parity at n=16; it classifies 99.3% of inputs correctly, and so behaviorally we conclude it has learned parity. Since the input-output pairs are fully enumerable, we can read out the learned function in its algebraic normal form (the monomial basis, via Mobius transform) and we get a degree-16 polynomial with ~25,000 monomials, whereas true parity is degree-1 with support 16. The network recovered the rule in every behaviorial sense, but the exact symbolic readout blows up due to a few hundred residual bit-errors pushed through the Mobius transform's non-locality.

The fix isn't a better network, but identifying a better basis to read out in. There are really three different questions hiding inside "did it recover the rule," and they coincide only at exactly zero error:

- **Behaviorial recovery:** agrees with rule on all but a small fraction of inputs.
- **Decoding recovery:** among all rules simple enough to be candidates, the target rule is the strictly nearest one; the behavior is closer to than to any other admissible rule, so it decodes to the target and nothing else.
- **Symbolic recovery:** the representation equals the rule's.

In the paper, we use two classical tools that are stable when symbolic equality is brittle:

- **Parseval's theorem in the Walsh-Hadamard basis:** the Walsh-Hadamard spectrum is the function rewritten in the parity basis, and that change of basis is an orthonormal rotation, so it preserves distances. Parseval makes it exact: the squared distance between the network's and the rule's spectra equals 4δ, four times the fraction of inputs they disagree on. So the spectral distance isn't a proxy for the bit-error rate, it is the bit-error rate, read as one continuous number. And because a rotation can't blow up a small perturbation, the measurement degrades linearly with error, where the Mobius/ANF transform sprays one bit-flip across many coefficients and explodes.
- **Reed-Muller decoding radius:** rules of a given degree sit far apart (any two differ on at least 2^(n−r) inputs) so a ball of half that radius around the target holds no other degree-r rule. If the network's truth table lands inside it, the target is provably the unique nearest degree-r rule: the behavior decodes to the rule, and a few hundred stray bit-errors don't flip the verdict because the ball has room to spare. A binary certificate, not a measurement; it needs only the rule's degree, which sets the radius; simpler rules give bigger balls and absorb more error.

So even in the most transparent setting, which basis we read out in decides whether we see the structure or a mess. That's why specifying the basis is the actual problem rather than a preliminary; we have to measure in the rule's own basis instead of reaching for a catch-all.

## Subthread - Grokking modular addition
If we want to study recovery, then we want a setting where we know the structure exists. Nanda et al.'s grokking transformer gives us this for free.

On modular addition, the embedding manifold Fourier participation ratio collapses by roughly 6.6x across the grokking transition, and does so as a leading ramp, before the sudden behavioral jump. The clean algebraic object (a circle) is visible in the weights *when we use the natural task basis*. This matters because, when recovery fails on harder tasks, we can't just say "the structure isn't there"; in this case it's cleanly measurable.

So this isn't just a story about late generalization, but also that grokking is a control condition for interpretability. In this case where the latent structure is known, the transition is visible, and the recovery question can be answered properly. Roughly, we can write this:

z_t(x) = z_{\text{rule},t}(x) + r_t(x)

Early on, the residual/memorization component carries the train behavior, and later, the rule component becomes stable and test accuracy snaps. In the Boolean case, we can examine this residual component directly: r = f XOR h, where f is the learned function and h is the target. The behavioral transition is a thresholded view of the underlying representational change. Here, grokking is about the dynamics of how a system moves from a long, high-complexity code to a short, structured one, with respect to the privileged basis.

## Subthread - Hopes on SAEs might be too high
*Coming soon*

## Subthread - Reference-free recovery up to gauge
*Coming soon*

## "Isn't this obvious? We can't identify the natural basis."
The anticipated objection: *of course we can't read out canonical structure without supplying a basis, that's just unsupervised representation learning being underdetermined*. Yes, it's impossible. But the impossibility isn't the binding constraint, and that's the part I think isn't obvious.

The Mobius noise zone has nothing to do with failing to find the basis: there we have the canonical basis, the rule is behaviorally recovered, and yet the readout still blows up. That brittleness is a different wall altogether, and is the one that bites first. Lumping it under the "can't identify the basis" category is a misdiagnosis.

Accepting the impossibility buys us things rather than ending the conversation. We can certify behavioral recovery against a supplied candidate without identifying anything (a question that practitioners actually have). The indeterminacy is a gauge quotient, so the invariant we can recover is exactly the one we needed; "non-identifiable" is not the same as "nothing survives." The residual is a structured signal we can act on. Whether we're at the statistical or computational wall is a measurement, not the end of the world.

Also, if "we can't identify the natural basis" were really treated as obvious, the field wouldn't be pushing for finding canonical SAE atoms and circuits, which are deliverables that quietly assume that basis-free identification is achievable. So either it isn't operationally obvious, or what follows from it (don't chase canonical recovery, certify behavioral recovery and read the residual instead) should already be standard, but it isn't.

The contribution isn't the impossiblity, it's that taking it seriously reorganizes the problem into more tractable questions, and a lot of good work is still solving the intractable version of it. Some of these moves aren't new on their own (certifying instead of discovering, the gauge quotient, both live in adjacent literatures); the claim is narrower, that the same structure recurs across all four subthreads, and that the operative failure is usually brittleness, not impossibility.

## Conclusions
1. **The failure is in the readout:** Sometimes the readout basis is brittle (the Mobius noise zone: a perfectly good solution detonates in the wrong basis), sometimes the readout is statistically underdetermined (SAE dilution at a Fano boundary; the gauge orbit). Both are cases of “the basis is the bottleneck,” and both tell us where to spend effort: choosing a stable basis, supplying structure, or accepting recovery-up-to-symmetry.
2. **Reconstruction is not recovery, and we often conflate them:** SAEs are a good reconstruction tool, but the geometric wall and dilution results prevent it from becoming an certificate of identifiability, and the field has discovered this gap. Identification needs a different, information-supplying step on top.
3. **Knowing when to abstain is a good thing:** A method that recovers structure *and tells us when it can’t* is worth more than one with a higher headline number and no idea where its blind spots are.

None of this gets us canonical, basis-free, unsupervised interpretability. The right basis is either brittle to read out exactly or underdetermined without side information, a fact at near-theorem status, and no amount of engineering can help us dig it out. The honest frontier could very well be finding methods that read out in a stable basis, reach the identifiability boundary, and are candid about living up to gauge, which is a smaller promise than “we’ll reverse engineer the network," but it's probably one we can safely keep.

## References
*To be completed*

### AI Disclaimer
*A large language model (LLM) was used in the drafting and editing of this content. The author reviewed and refined the text to ensure accuracy and maintain personal voice.*



