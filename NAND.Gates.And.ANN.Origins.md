## OpenAI ChatGPT v1.2025.232 

**Q: NAND gates. Because nand gates implemented in hardware can represent formal language and reasoning expressed through software -- was this the insight to develop an artificial neural network architectural equivalent of this mathematical/reasoning model?**

**A:** Short answer: No. The “NAND is universal” insight predates neural nets and did not drive their design. Early ANN theory came from McCulloch–Pitts neurons as threshold logic plus biological inspiration; later results showed approximation and computation power well beyond Boolean gate emulation. ([Wikipedia][1], [California State University Long Beach][2], [Information Services and Technology][3], [cognitivemedium.com][4], [binds.cs.umass.edu][5])

What’s true:

* NAND/NOR functional completeness was established in classical logic (Sheffer, 1913). Any Boolean function can be built from NAND alone. This is switching theory, not ANN architecture. ([Wikipedia][1], [ProofWiki][6])
* McCulloch & Pitts (1943) modeled neurons as binary threshold units implementing AND/OR/NOT and showed networks define a logical calculus of nervous activity. This set the logic–computation framing for later ANNs. ([California State University Long Beach][2])
* Rosenblatt’s perceptron (1958–60) operationalized trainable threshold units; Minsky & Papert (1969) characterized single-layer limits (e.g., XOR). These works were about learnability and geometry, not NAND per se. ([Wikipedia][7])
* Modern theory: universal approximation theorems (Cybenko 1989; Hornik 1989) show shallow networks with suitable nonlinearities approximate any continuous function on compact sets; recurrent nets are Turing-complete (Siegelmann & Sontag, 1995). This is a functional-analysis and computation-theory basis, not gate universality. ([Information Services and Technology][3], [cognitivemedium.com][4], [binds.cs.umass.edu][5])

Useful equivalence to know:

* A single perceptron implements NAND since NAND is linearly separable. One concrete realization: inputs $x_1,x_2\in\{0,1\}$, weights $w_1=w_2=-1$, bias $b=1.5$, output $y=\mathbf{1}[w_1x_1+w_2x_2+b>0]$. This yields $y=1$ except when $x_1=x_2=1$. AND/OR are also linearly separable; XOR is not. For counts: with two Boolean inputs, 14 of 16 functions are linearly separable. ([Wikipedia][8])

Bottom line: ANNs can emulate NAND and full Boolean circuits, but the field’s core motivation and theory grew from threshold-logic neurobiology and, later, universal approximation and computational-power results—not from the NAND insight itself. ([California State University Long Beach][2], [Information Services and Technology][3], [cognitivemedium.com][4])

```
Sources (textual links)
- Sheffer stroke (functional completeness): https://en.wikipedia.org/wiki/Sheffer_stroke
- NAND logic (functional completeness): https://en.wikipedia.org/wiki/NAND_logic
- Proof that NAND/NOR are functionally complete: https://proofwiki.org/wiki/NAND_and_NOR_are_Functionally_Complete
- McCulloch & Pitts (1943) “A Logical Calculus of the Ideas Immanent in Nervous Activity”: https://home.csulb.edu/~cwallis/382/readings/482/mccolloch.logical.calculus.ideas.1943.pdf
- Perceptron overview and history: https://en.wikipedia.org/wiki/Perceptron
- Minsky & Papert (1969) Perceptrons: https://en.wikipedia.org/wiki/Perceptrons_%28book%29
- Cybenko (1989) universal approximation: https://web.njit.edu/~usman/courses/cs677/10.1.1.441.7873.pdf
- Hornik (1989) universal approximation: https://cognitivemedium.com/magic_paper/assets/Hornik.pdf
- Siegelmann & Sontag (1995) neural nets are Turing-complete: https://binds.cs.umass.edu/papers/1995_Siegelmann_JComSysSci.pdf
- Linear separability of Boolean functions (counts): https://en.wikipedia.org/wiki/Linear_separability
```

[1]: https://en.wikipedia.org/wiki/Sheffer_stroke?utm_source=chatgpt.com "Sheffer stroke"
[2]: https://home.csulb.edu/~cwallis/382/readings/482/mccolloch.logical.calculus.ideas.1943.pdf?utm_source=chatgpt.com "A logical calculus of the ideas immanent in nervous activity"
[3]: https://web.njit.edu/~usman/courses/cs677/10.1.1.441.7873.pdf?utm_source=chatgpt.com "Approximation by superpositions of a sigmoidal function"
[4]: https://cognitivemedium.com/magic_paper/assets/Hornik.pdf?utm_source=chatgpt.com "Multilayer Feedforward Networks are Universal ..."
[5]: https://binds.cs.umass.edu/papers/1995_Siegelmann_JComSysSci.pdf?utm_source=chatgpt.com "On the Computational Power of Neural Nets"
[6]: https://proofwiki.org/wiki/NAND_and_NOR_are_Functionally_Complete?utm_source=chatgpt.com "NAND and NOR are Functionally Complete"
[7]: https://en.wikipedia.org/wiki/Perceptron?utm_source=chatgpt.com "Perceptron"
[8]: https://en.wikipedia.org/wiki/Linear_separability?utm_source=chatgpt.com "Linear separability"

**Q: Can an ANN which emulates only NAND be arranged into an architecture which matches or surpasses the GPT architecture?**

**A:** Answer: In principle, yes (expressivity). In practice, no (training, accuracy, and efficiency). A network restricted to NAND can emulate any finite-precision Transformer/GPT forward pass, but current evidence shows binarized or Booleanized models lag full-precision GPT-class models on accuracy and data efficiency, unless you relax “only NAND” during training or add hybrid analog/digital math blocks. ([Wikipedia][1], [arXiv][2], [Journal of Machine Learning Research][3])

Key points

1. What “ANN that emulates only NAND” reduces to
   A pure-NAND ANN is just a Boolean circuit family; by functional completeness, NAND alone can implement any Boolean function. This aligns with circuit-complexity class P/poly (polynomial-size circuit families). So representationally you can implement any fixed-length computation, including a Transformer at chosen bit-width. ([Wikipedia][4], [Internet Encyclopedia of Philosophy][5])

2. Compiling GPT into NAND is straightforward in theory
   Matrix multiplies → adders + multipliers; softmax/exponentials and layer-norm → fixed-point math blocks (e.g., LUT/PLA approximations, CORDIC-style kernels), all reducible to NAND gates. Numerous softmax/exponential hardware designs exist, confirming digital implementability. ([Wikipedia][6], [lca.ece.utexas.edu][7], [arXiv][8], [MDPI][9], [Nature][10])

3. But training breaks if you insist on NAND everywhere
   Backprop needs gradients; Boolean NAND is non-differentiable. Practical binary networks use surrogate gradients such as the straight-through estimator (STE) or continuous relaxations (Gumbel-Softmax/Concrete). These methods work but introduce bias/instability and typically underperform full precision on hard NLP tasks. ([arXiv][11])

4. What we actually know from binarized models
   • Vision: XNOR-Net reported \~58× convolution speedup on CPUs and \~32× weight-memory reduction, with accuracy close to full precision on ImageNet for some backbones. Good for efficiency, not a path to new SOTA accuracy.&#x20;
   • NLP: BinaryBERT achieved \~24× smaller models with only a “slight” drop on GLUE/SQuAD via a careful ternary-to-binary training recipe, but still below full-precision baselines. Fully binarized BERTs (e.g., BiBERT) mitigate loss with specialized tricks; they target compression, not surpassing SOTA. ([ACL Anthology][12], [OpenReview][13])

5. Depth/size constraints matter
   Shallow constant-depth NAND circuits (AC⁰) cannot compute even parity with polynomial size; many sequence tasks need long-range interactions, which forces depth or size blowups in a NAND-only view. Transformers side-step this with real-valued attention and continuous optimization. ([wiki.epfl.ch][14])

6. Expressivity of Transformers vs NAND
   Transformers are universal approximators for sequence-to-sequence maps under mild conditions and can simulate very general computations given appropriate precision/positional mechanisms; some results prove attention is Turing-complete under idealized settings. These show GPT-like models already saturate expressivity without reverting to gate-level design. ([arXiv][2], [OpenReview][15], [Journal of Machine Learning Research][3])

Bottom line
* Capacity: A NAND-only ANN can represent any fixed-precision GPT. So “match” is possible in theory. ([Wikipedia][1])
* Surpass: No empirical evidence. Binary/bool variants trade accuracy and data efficiency for memory/throughput. State-of-the-art LLMs rely on higher precision and differentiable training to reach current performance. ([ACL Anthology][12], [OpenReview][13])
* Practical path: If you want NAND-like efficiency, train a standard Transformer, then quantize or binarize selectively, and use hardware-friendly approximations for softmax/layer-norm. Full “only NAND” at train- and run-time is theoretically clean but empirically inferior today. ([arXiv][16], [MDPI][9])

Textual sources

* “NAND logic” — en.wikipedia.org/wiki/NAND\_logic ([Wikipedia][4])
* “Sheffer stroke (functional completeness)” — en.wikipedia.org/wiki/Sheffer\_stroke ([Wikipedia][17])
* “Boolean circuit” and P/poly — en.wikipedia.org/wiki/Boolean\_circuit ; en.wikipedia.org/wiki/P/poly ([Wikipedia][18])
* “Parity not in AC⁰” — Furst, Saxe, Sipser (1984) — wiki.epfl.ch/.../Furst%20Saxe%20Sipser%20-1984-...pdf ([wiki.epfl.ch][14])
* “Transformers are universal approximators” — arxiv.org/abs/1912.10077 ; openreview\.net/pdf?id=ByxRM0Ntvr ([arXiv][2], [OpenReview][15])
* “Attention is Turing Complete” — jmlr.org/papers/volume22/20-302/20-302.pdf ([Journal of Machine Learning Research][3])
* “STE for discrete/binary nets” — arxiv.org/abs/1308.3432 ; arxiv.org/abs/1903.05662 ([arXiv][11])
* “Gumbel-Softmax/Concrete” — arxiv.org/abs/1611.00712 ([arXiv][19])
* “XNOR-Net” — ai2-website.s3.amazonaws.com/publications/XNOR-Net.pdf&#x20;
* “BinaryBERT” — aclanthology.org/2021.acl-long.334.pdf ([ACL Anthology][12])
* “BiBERT” — openreview\.net/pdf?id=5xEgrl\_5FAJ ([OpenReview][13])
* “Hardware softmax/exponential approximations” — arxiv.org/pdf/2311.13290 ; mdpi.com/2079-9292/14/12/2337 ; nature.com/articles/s41598-021-94691-7 ; lca.ece.utexas.edu/pubs/SoftMax\_\_\_ASAP2020\_June14.pdf ([arXiv][8], [MDPI][9], [Nature][10], [lca.ece.utexas.edu][7])

If you want, I can sketch a compile-to-NAND pipeline for a small Transformer block: fixed-point spec, gate-count estimates for matmul/softmax, and the training recipe (STE + distillation + selective dequant).

[1]: https://en.wikipedia.org/wiki/P/poly?utm_source=chatgpt.com "P/poly"
[2]: https://arxiv.org/abs/1912.10077?utm_source=chatgpt.com "Are Transformers universal approximators of sequence-to- ..."
[3]: https://jmlr.org/papers/volume22/20-302/20-302.pdf?utm_source=chatgpt.com "Attention is Turing Complete"
[4]: https://en.wikipedia.org/wiki/NAND_logic?utm_source=chatgpt.com "NAND logic"
[5]: https://iep.utm.edu/sheffers/?utm_source=chatgpt.com "The Sheffer Stroke"
[6]: https://en.wikipedia.org/wiki/Binary_multiplier?utm_source=chatgpt.com "Binary multiplier"
[7]: https://lca.ece.utexas.edu/pubs/SoftMax___ASAP2020_June14.pdf?utm_source=chatgpt.com "Design Space Exploration for Softmax Implementations"
[8]: https://arxiv.org/pdf/2311.13290?utm_source=chatgpt.com "Hyft: A Reconfigurable Softmax Accelerator with Hybrid ..."
[9]: https://www.mdpi.com/2079-9292/14/12/2337?utm_source=chatgpt.com "Hardware Accelerator for Approximation-Based Softmax ..."
[10]: https://www.nature.com/articles/s41598-021-94691-7?utm_source=chatgpt.com "A pseudo-softmax function for hardware-based high speed ..."
[11]: https://arxiv.org/abs/1308.3432?utm_source=chatgpt.com "Estimating or Propagating Gradients Through Stochastic ..."
[12]: https://aclanthology.org/2021.acl-long.334.pdf "BinaryBERT: Pushing the Limit of BERT Quantization"
[13]: https://openreview.net/pdf?id=5xEgrl_5FAJ&utm_source=chatgpt.com "BIBERT: ACCURATE FULLY BINARIZED BERT"
[14]: https://wiki.epfl.ch/edicpublic/documents/Candidacy%20exam/Furst%20Saxe%20Sipser%20-%201984%20-%20Parity%20circuits%20and%20the%20polynomial-time%20hierarchy.pdf?utm_source=chatgpt.com "Parity, circuits, and the polynomial-time hierarchy"
[15]: https://openreview.net/pdf?id=ByxRM0Ntvr&utm_source=chatgpt.com "ARE TRANSFORMERS UNIVERSAL APPROXIMATORS"
[16]: https://arxiv.org/pdf/2103.09301?utm_source=chatgpt.com "Softermax: Hardware/Software Co-Design of an Efficient ..."
[17]: https://en.wikipedia.org/wiki/Sheffer_stroke?utm_source=chatgpt.com "Sheffer stroke"
[18]: https://en.wikipedia.org/wiki/Boolean_circuit?utm_source=chatgpt.com "Boolean circuit"
[19]: https://arxiv.org/abs/1611.00712?utm_source=chatgpt.com "The Concrete Distribution: A Continuous Relaxation of Discrete Random Variables"

**Q: Because backprop needs gradients, what if the ANN nand model is realized in a (step-wise) continuous way which supports this requirement; or perhaps a constrained slack-variable is supplied as an add-in?**

**A:** Yes. Your two ideas are exactly the standard fixes: (1) a differentiable “soft-NAND” with temperature/annealing so backprop works, and (2) constrained training with slack variables via augmented-Lagrangian or ADMM. Both are practical. They train. They still face gradient-mismatch and saturation issues. Empirically they yield compact, fast models, but accuracy typically trails full-precision Transformers unless you add distillation and careful schedules. ([arXiv][1], [Proceedings of Machine Learning Research][2])

Design 1 — continuous soft-NAND (temperature, annealing)
Define inputs $x_1,x_2\in[0,1]$. Use a logistic “soft step” to emulate NAND with margin:

$$
\tilde{y}_{\tau}=\sigma\!\left(\frac{1.5 - x_1 - x_2}{\tau}\right),\quad
\sigma(t)=\frac{1}{1+e^{-t}}
$$

As $\tau\!\downarrow\!0$, $\tilde{y}_{\tau}$ approaches the hard NAND Heaviside; with $\tau>0$ it is smooth so $\partial \tilde{y}_{\tau}/\partial x$ exists for backprop. Train with cross-entropy to the NAND truth table plus a binarization regularizer $R(y)=\sum_i \tilde{y}_i(1-\tilde{y}_i)$. Anneal $\tau$ from, e.g., 1.0 → 0.05 over epochs so gradients do not vanish early and discreteness sharpens late. This is the same “soft-to-hard” trick used for discrete relaxations. ([MathWorld][3], [Wikipedia][4], [ScienceDirect][5], [arXiv][6])

Design 2 — slack-variable constraints (augmented Lagrangian / ADMM)
Introduce an auxiliary binary variable $z\in\{0,1\}$ per gate and constrain $z\approx \tilde{y}_{\tau}$. Optimize

$$
\min_\theta \; \mathcal{L}_{\text{task}}(\theta) \;+\; \sum_i \big[\lambda\|z_i-\tilde{y}_{\tau,i}\|_2^2+\mu\,s_i\big]
\quad\text{s.t.}\quad z_i\in\{0,1\},\; s_i\ge 0
$$

with an augmented-Lagrangian or ADMM loop: θ-update (SGD on smooth surrogate), z-update (projection toward $\{0,1\}$ with a proximal step, optionally with a small trust region), dual update; increase penalty $\lambda$ over time. This “split” yields stable training of discrete constraints and is standard in quantization literature. ([Now Publishers][7], [Stanford University][8], [Proceedings of Machine Learning Research][2])

Training recipe that works in practice

1. Start continuous: initialize with full-precision weights; train gates as $\tilde{y}_{\tau}$ with $\tau\!=\!0.5\text{–}1.0$. Use STE only if you snap any variable to $\{0,1\}$ during the forward pass. ([arXiv][9])
2. Truth-table curriculum: mix the four NAND corner cases with small Gaussian jitter so margins are learned; optimize cross-entropy to targets $[1,1,1,0]$. Sigmoid-Heaviside approximation theory supports this setup. ([biomath.bg][10], [Semantic Scholar PDFs][11])
3. Anneal and project: decay $\tau$ geometrically; in late epochs apply a proximal “snap” to $\{0,1\}$ each step (ProxQuant-style) or an ADMM z-projection. ([arXiv][12], [OpenReview][13])
4. Distill from a teacher Transformer to recover accuracy lost to binarization. Use layerwise MSE on hidden states and logits. ([ACL Anthology][14])
5. If you hard-binarize weights/acts, keep a real-valued shadow copy for gradient accumulation (BinaryConnect/BNN practice). ([arXiv][15])
6. Optional discrete choice: replace some hard decisions with Gumbel-Softmax or Concrete during training; take argmax at inference. ([arXiv][16])

What to expect empirically
* Efficiency: fully binarized CNNs show 32× memory savings and up to \~58× faster convolutions; binarized Transformers report \~31× model-size and \~56× FLOP reductions. These numbers are architecture-dependent but are typical order-of-magnitude wins. ([arXiv][17], [ai2-website.s3.amazonaws.com][18])
* Accuracy: with careful tricks, fully binarized BERT variants can outperform other quantized baselines, yet they are usually below strong full-precision models trained at scale; specialized distillation and attention redesign help but do not guarantee parity. ([arXiv][19], [OpenReview][20])
* Stability: STE works but is biased; proximal/ADMM routes provide a principled alternative for discrete constraints. Expect sensitivity to $\tau$, penalty schedules, and learning rates. ([arXiv][1], [Proceedings of Machine Learning Research][2])

Minimal soft-NAND block you can drop in
Forward: $\tilde{y}_{\tau}=\sigma((1.5 - x_1 - x_2)/\tau)$. Loss: $\mathcal{L}=\mathcal{L}_{\text{task}}+\alpha\,\text{CE}(\tilde{y}_{\tau},\text{NAND})+\beta\sum_i \tilde{y}_i(1-\tilde{y}_i)$. Schedule: $\tau: 1.0\!\to\!0.1$, $\alpha: 1\!\to\!4$, $\beta: 0\!\to\!0.1$. Optional hardening: STE on $y=\mathbf{1}[\tilde{y}_{\tau}>0.5]$ for the last 10–20% of training. ([Wikipedia][4], [arXiv][9])

Bottom line
Your “step-wise continuous NAND” and “slack-variable add-in” are the two known paths to make NAND-style networks trainable with backprop. They are feasible and already in use under the names surrogate gradients, Concrete/Gumbel-Softmax, proximal regularization, and ADMM quantization. They can match throughput and memory goals. Surpassing GPT-class accuracy at equal scale remains unproven. ([arXiv][9], [Proceedings of Machine Learning Research][2])

References (textual links)
* Bengio et al., “Estimating or Propagating Gradients Through Stochastic Neurons,” 2013: [https://arxiv.org/abs/1308.3432](https://arxiv.org/abs/1308.3432) ([arXiv][9])
* Courbariaux et al., “BinaryConnect,” 2015: [https://arxiv.org/abs/1511.00363](https://arxiv.org/abs/1511.00363) ; Hubara et al., “BNN,” 2016: [https://arxiv.org/abs/1602.02830](https://arxiv.org/abs/1602.02830) ([arXiv][15])
* Maddison et al., “Concrete Distribution,” 2016: [https://arxiv.org/abs/1611.00712](https://arxiv.org/abs/1611.00712) ; Jang et al., “Gumbel-Softmax,” 2017 overview: [https://casmls.github.io/general/2017/02/01/GumbelSoftmax.html](https://casmls.github.io/general/2017/02/01/GumbelSoftmax.html) ([arXiv][6], [casmls.github.io][21])
* Bai et al., “ProxQuant,” 2018: [https://arxiv.org/abs/1810.00861](https://arxiv.org/abs/1810.00861) ; OpenReview version: [https://openreview.net/pdf?id=HyzMyhCcK7](https://openreview.net/pdf?id=HyzMyhCcK7) ([arXiv][12], [OpenReview][13])
* Huang et al., “ADMM-Q,” AISTATS 2021: [https://proceedings.mlr.press/v130/huang21a/huang21a.pdf](https://proceedings.mlr.press/v130/huang21a/huang21a.pdf) ; Boyd et al., ADMM tutorial, 2011: [https://web.stanford.edu/\~boyd/papers/pdf/admm\_distr\_stats.pdf](https://web.stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf) ([Proceedings of Machine Learning Research][2], [Stanford University][8])
* Rastegari et al., “XNOR-Net,” 2016: [https://arxiv.org/abs/1603.05279](https://arxiv.org/abs/1603.05279) ; PDF: [https://ai2-website.s3.amazonaws.com/publications/XNOR-Net.pdf](https://ai2-website.s3.amazonaws.com/publications/XNOR-Net.pdf) ([arXiv][17], [ai2-website.s3.amazonaws.com][18])
* Bai et al., “BinaryBERT,” ACL 2021: [https://aclanthology.org/2021.acl-long.334.pdf](https://aclanthology.org/2021.acl-long.334.pdf) ; Qin et al., “BiBERT,” ICLR 2022: [https://arxiv.org/abs/2203.06390](https://arxiv.org/abs/2203.06390) ([ACL Anthology][14], [arXiv][19])
* Survey white paper on quantization (PTQ/QAT): Nagel et al., 2021: [https://arxiv.org/pdf/2106.08295](https://arxiv.org/pdf/2106.08295) ([arXiv][22])
* Sigmoid and step-approximation background: Wolfram MathWorld sigmoid and Heaviside pages: [https://mathworld.wolfram.com/SigmoidFunction.html](https://mathworld.wolfram.com/SigmoidFunction.html) , [https://mathworld.wolfram.com/HeavisideStepFunction.html](https://mathworld.wolfram.com/HeavisideStepFunction.html) ([MathWorld][3])

1]: https://arxiv.org/abs/1903.05662?utm_source=chatgpt.com "Understanding Straight-Through Estimator in Training Activation ..."  
2]: https://proceedings.mlr.press/v130/huang21a/huang21a.pdf?utm_source=chatgpt.com "Alternating Direction Method of Multipliers for Quantization"  
3]: https://mathworld.wolfram.com/SigmoidFunction.html?utm_source=chatgpt.com "Sigmoid Function -- from Wolfram MathWorld"  
4]: https://en.wikipedia.org/wiki/Sigmoid_function?utm_source=chatgpt.com "Sigmoid function - Wikipedia"  
5]: https://www.sciencedirect.com/science/article/abs/pii/S0378475415002554?utm_source=chatgpt.com "On the approximation of the step function by some sigmoid ..."  
6]: https://arxiv.org/abs/1611.00712?utm_source=chatgpt.com "A Continuous Relaxation of Discrete Random Variables"  
7]: https://www.nowpublishers.com/article/Details/MAL-016?utm_source=chatgpt.com "Distributed Optimization and Statistical Learning via the ..."  
8]: https://web.stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf?utm_source=chatgpt.com "Distributed Optimization and Statistical Learning via the ..."  
9]: https://arxiv.org/abs/1308.3432?utm_source=chatgpt.com "Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation"  
10]: https://www.biomath.bg/s%20markov%20publications/A2015-MathChem_Hausdorff_Dist.pdf?utm_source=chatgpt.com "On the Hausdorff distance between the Heaviside step ..."  
11]: https://pdfs.semanticscholar.org/72a5/76a56814bbdfdf77680974df80ca989d4290.pdf?utm_source=chatgpt.com "On the Approximation of the Cut and Step Functions by ..."  
12]: https://arxiv.org/abs/1810.00861?utm_source=chatgpt.com "ProxQuant: Quantized Neural Networks via Proximal Operators"  
13]: https://openreview.net/pdf?id=HyzMyhCcK7&utm_source=chatgpt.com "PROXQUANT: QUANTIZED NEURAL NETWORKS VIA ..."  
14]: https://aclanthology.org/2021.acl-long.334.pdf?utm_source=chatgpt.com "BinaryBERT: Pushing the Limit of BERT Quantization"  
15]: https://arxiv.org/abs/1511.00363?utm_source=chatgpt.com "BinaryConnect: Training Deep Neural Networks with binary weights during propagations"  
16]: https://arxiv.org/pdf/1611.00712?utm_source=chatgpt.com "The Concrete distribution"  
17]: https://arxiv.org/abs/1603.05279?utm_source=chatgpt.com "XNOR-Net: ImageNet Classification Using Binary Convolutional Neural Networks"  
18]: https://ai2-website.s3.amazonaws.com/publications/XNOR-Net.pdf?utm_source=chatgpt.com "XNOR-Net: ImageNet Classification Using Binary ..."  
19]: https://arxiv.org/abs/2203.06390?utm_source=chatgpt.com "BiBERT: Accurate Fully Binarized BERT"  
20]: https://openreview.net/pdf?id=5xEgrl_5FAJ&utm_source=chatgpt.com "BIBERT: ACCURATE FULLY BINARIZED BERT"  
21]: https://casmls.github.io/general/2017/02/01/GumbelSoftmax.html?utm_source=chatgpt.com "The Gumbel-Softmax Trick for Inference of Discrete Variables"  
22]: https://arxiv.org/pdf/2106.08295?utm_source=chatgpt.com "A White Paper on Neural Network Quantization"  
