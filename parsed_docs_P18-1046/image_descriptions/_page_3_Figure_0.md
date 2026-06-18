# _page_3_Figure_0.jpg

**Source:** `assets/_page_3_Figure_0.jpg`

**Generated:** 2026-05-30 09:54:53

---

The image is a diagram illustrating a Generative Adversarial Network (GAN) framework, specifically designed for a domain-specific (DS) positive dataset. It includes three main components: the Generator (G), the Discriminator (D), and a Sampling process. The Generator takes a batch of samples from the DS positive dataset and outputs a set of probabilities (p1 to pn) for each sample. The Sampling process then categorizes these samples into high-confidence and low-confidence groups based on these probabilities. The high-confidence samples are labeled as 0 and fed into the Discriminator, while the low-confidence samples are labeled as 1 and also fed into the Discriminator. The Discriminator is trained to distinguish between real and generated samples, with its parameters being updated based on the feedback from the Generator and the Sampling process. The diagram also shows a pre-training phase for the Discriminator using the DS positive and negative datasets.