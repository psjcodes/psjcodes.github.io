---
title: On Modular Addition, Weight Decay Slowly Builds a Fourier Bandstop
created: 2026-06-23
modified: 2026-06-23
---

This note is a subthread of [[thread-basis-bottleneck|the basis-is-the-bottleneck]] thread, and the intervention counterpart to [[grok-fourier-pr|the participation-ratio]] note: there I showed the embedding's frequency concentration *leads* the grok; here I force that concentration and see whether it *causes* a faster grok. On modular addition it does, by about 5x, and the lever turns out to be frequency-domain norm-shedding specifically; weight decay reaches the same concentrated spectrum, but slowly, through a more uniform shedding.

## Setup

I train Nanda's 1-layer transformer from scratch: `a + b mod 113`, 30% of the pairs as train, AdamW (`lr = 1e-3`, `wd = 1`, betas `(0.9, 0.98)`), 16k epochs, 2 seeds. I mark the grok as the steepest drop in `-log(test loss)`, which is the same definition the PR note uses. The question is whether an explicit push toward a sparse frequency spectrum beats the implicit cleanup that weight decay does on its own. Using the same transformer means the intervention here and the observation in the PR note sit on the same model and task.

## The regularizer and the two controls

The grokked solution is sparse in the Fourier basis (a few key modes = the circle[2]); memorization is broadband. So the natural intervention is **Fourier-L1**: an L1 penalty on the DFT (over the symbol axis) of the embedding `W_E`, the *same* `E` whose participation ratio the PR note tracks. L1 in the frequency domain shrinks everything but drives the small broadband coefficients to zero while keeping the large key modes; it is a magnitude bandstop that *also* sheds norm.

I ran two controls to isolate what Fourier-L1 actually buys:

- **2xwd:** double the weight decay; uniform shrinkage. If Fourier-L1 is just "more regularization," this should match it.
- **weight-L1:** an L1 penalty of the *same init magnitude*, but in the weight domain instead of the frequency domain. If any L1 sparsity will do, this should match Fourier-L1; if the frequency domain specifically matters, it won't.

I set each penalty by its *initial magnitude*: its size as a fraction of the training loss, i.e. a regularization strength, rather than a raw coefficient, since the coefficient is not comparable across the two domains (L1 of DFT(E) ~ 6x L1 of E at init). The strong Fourier-L1 and weight-L1 are matched at magnitude 0.3, so the *domain* is the only difference between them; the gentle Fourier-L1 sits at 0.1.

## Result

![Test loss (log scale) across the arms; Fourier-L1 groks earliest](grok-fourier-bandstop.png)

Epochs-to-grok (steepest drop in `-log` test loss), Nanda transformer, `p = 113`, 2 seeds:

| arm | epochs-to-grok | vs baseline |
|---|---|---|
| baseline (wd=1) | 8000 | 1.00x |
| 2xwd | 3700 | 2.16x |
| weight-L1 (penalty 0.3) | 5100 | 1.57x |
| Fourier-L1 (penalty 0.1) | 1400 | 5.71x |
| Fourier-L1 (penalty 0.3) | 1600 | 5.00x |

- **Fourier-L1 beats 2xwd (~5x vs 2.16x):** frequency-aware sparsity is not just more shrinkage.
- **Fourier-L1 beats weight-L1 at matched penalty magnitude (~5x vs 1.57x):** the *frequency domain* is what matters, not L1 sparsity in general.

So the speedup is specifically about shedding norm in the Fourier basis. The two penalty strengths land at about the same place (5.71x and 5.00x, within noise at 2 seeds), so I do not read a clean dose-response from this run. I also ran it on an MLP version of the same task (`a + b mod 31`, ~4.6x), so the effect is not an artifact of one architecture.

## What this says

On this task, weight decay slowly builds a Fourier bandstop: uniform shrinkage is the mechanism, and over training it starves the broadband memorization modes while the key Fourier modes survive, so the embedding's spectrum ends up bandstop-shaped. Applying that bandstop directly in the frequency domain reaches the same place far faster. Paired with the PR note, the picture is symmetric: concentration in the privileged basis *leads* the grok (observation), and *forcing* that concentration (specifically, frequency-domain norm-shedding) *causes* a faster grok (intervention).

## Caveats

- **One task, one architecture, two seeds:** This is `a + b mod 113` on Nanda's 1-layer transformer. That weight decay slowly builds a Fourier bandstop is a statement about *this* setting, where the privileged basis is the Fourier basis the cyclic group hands us; I make no claim about weight decay in general.
- **It presupposes the basis, like the PR note:** The whole intervention needs the Fourier axis; on a task without a known symmetry you could not write down Fourier-L1 in the first place.
- **Measurement details:** Epochs-to-grok is the steepest drop in `-log(test loss)`, windowed to the grokking transition (the event the PR note's full-run argmax captures); on the full noisy curve a late low-loss spike can otherwise hijack the unwindowed argmax. Training is float32 plain cross-entropy (MPS, no float64 high-precision loss), 2 seeds, so the absolute baseline grok (~8,000) differs from the PR note's checkpoint (~10,700) by training run, not by threshold, and the post-grok tails oscillate (cleanup-phase noise) rather than settling smoothly.

## References
1. Alethea Power, Yuri Burda, Harri Edwards, Igor Babuschkin, Vedant Misra. *Grokking: Generalization Beyond Overfitting on Small Algorithmic Datasets.* arXiv:2201.02177, 2022. [arXiv:2201.02177](https://arxiv.org/abs/2201.02177)
2. Neel Nanda, Lawrence Chan, Tom Lieberum, Jess Smith, Jacob Steinhardt. *Progress Measures for Grokking via Mechanistic Interpretability.* ICLR 2023. [arXiv:2301.05217](https://arxiv.org/abs/2301.05217)

### AI Disclaimer
*A large language model (LLM) was used in the drafting and editing of this content. The author reviewed and refined the text to ensure accuracy and maintain personal voice.*
