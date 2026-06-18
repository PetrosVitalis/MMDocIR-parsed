<span id="page-0-0"></span>
# Chameleon: Mixed-Modal Early-Fusion Foundation Models

Chameleon Team1,∗

1FAIR at Meta

∗See Contributions section for full author list.

We present Chameleon, a family of early-fusion token-based mixed-modal models capable of understanding and generating images and text in any arbitrary sequence. We outline a stable training approach from inception, an alignment recipe, and an architectural parameterization tailored for the early-fusion, token-based, mixed-modal setting. The models are evaluated on a comprehensive range of tasks, including visual question answering, image captioning, text generation, image generation, and long-form mixed modal generation. Chameleon demonstrates broad and general capabilities, including state-of-the-art performance in image captioning tasks, outperforms Llama-2 in text-only tasks while being competitive with models such as Mixtral 8x7B and Gemini-Pro, and performs non-trivial image generation, all in a single model. It also matches or exceeds the performance of much larger models, including Gemini Pro and GPT-4V, according to human judgments on a new long-form mixed-modal generation evaluation, where either the prompt or outputs contain mixed sequences of both images and text. Chameleon marks a significant step forward in a unified modeling of full multimodal documents.

Date: May 17, 2024


## 1 Introduction

Recent multimodal foundation models are very widely adopted but still model different modalities separately, often using modality specific encoders or decoders. This can limit their ability to integrate information across modalities and generate multimodal documents that can contain arbitrary sequences of images and text. In this paper, we present Chameleon, a family of mixed-modal foundation models capable of generating and reasoning with mixed sequences of arbitrarily interleaved textual and image content (Figures 2-4). This allows for full multimodal document modeling, which is a direct generalization of standard multimodal tasks such as image generation, understanding and reasoning over images, and text-only LLMs. Chameleon is instead designed to be mixed-model from inception and uses a uniform architecture trained from scratch in an end-to-end fashion on an interleaved mixture of all modalities, i.e., images, text, and code.

Our unified approach uses fully token-based representations for both image and textual modalities (Figure 1). By quantizing images into discrete tokens, analogous to words in text, we can apply the same transformer architecture to sequences of both image and text tokens, without the need for separate image/text encoders (Alayrac et al., 2022; Liu et al., 2023b; Laurençon et al., 2023) or domain-specific decoders (Ramesh et al., 2022; Jin et al., 2023; Betker et al., 2023). This early-fusion approach, where all modalities are projected into a shared representational space from the start, allows for seamless reasoning and generation across modalities. However, it also presents significant technical challenges, particularly in terms of optimization stability and scaling.

We address these challenges through a combination of architectural innovations and training techniques. We introduce novel modifications to the transformer architecture, such as query-key normalization and revised placement of layer norms, which we find to be crucial for stable training in the mixed-modal setting (Section 2.3). We further show how to adapt the supervised finetuning approaches used for text-only LLMs to the mixed-modal setting, enabling strong alignment at scale (Section 3). Using these techniques, we successfully train Chameleon-34B on 5x the number of tokens as Llama-2 – enabling new mixed-modal applications while still matching or even outperforming existing LLMs on unimodal benchmarks.

<span id="page-1-0"></span>
Figure 1 Chameleon represents all modalities — images, text, and code, as discrete tokens and uses a uniform transformer-based architecture that is trained from scratch in an end-to-end fashion on ∼10T tokens of interleaved mixed-modal data. As a result, Chameleon can both reason over, as well as generate, arbitrary mixed-modal documents. Text tokens are represented in green and image tokens are represented in blue.
![](assets/_page_1_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_1_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:19:23
> 
> ---
> 
> The image contains a flowchart-style diagram illustrating a mixed-modal auto-regressive language model (LM) for pre-training and generation tasks. The diagram is divided into two main sections: (a) Mixed-Modal Pre-Training and (b) Mixed-Modal Generation. 
> 
> In the pre-training section, the flowchart shows a text prompt and an image prompt being processed by an Image Tokenizer, which then feeds into a Mixed-Modal Auto-Regressive LM. The output includes start and end images, as well as text and image outputs. The text output is "Here is a recipe for banana bread," and the image output is a sliced loaf of banana bread.
> 
> In the generation section, the flowchart shows the same Mixed-Modal Auto-Regressive LM processing a text prompt and an image prompt. The output again includes start and end images, as well as text and image outputs. The text output is "What can I bake with this?" and the image output is a bowl of bananas and a banana.
> 
> The diagram highlights the integration of text and image data in the pre-training and generation processes of the language model, demonstrating how the model can generate both text and image outputs based on the provided prompts.


Extensive evaluations demonstrate that Chameleon is a broadly capable model on a diverse set of tasks. On visual question answering and image captioning benchmarks, Chameleon-34B achieves state-of-the-art performance, outperforming models like Flamingo, IDEFICS and Llava-1.5 (Section 5.2). At the same time, it maintains competitive performance on text-only benchmarks, matching models like Mixtral 8x7B and Gemini-Pro on commonsense reasoning and reading comprehension tasks (Section 5.1). But perhaps most impressively, Chameleon unlocks entirely new capabilities in terms of mixed-modal reasoning and generation.

As using only static, public benchmarks to evaluate model performance could be limited (Schaeffer, 2023), we also conduct a carefully designed human evaluation experiment by measuring the quality of mixed-modal long form responses to open-ended prompts. Chameleon-34B substantially outperforms strong baselines like Gemini-Pro and GPT-4V (Section 4), achieving a 60.4% preference rate against Gemini-Pro and a 51.6% preference rate against GPT-4V in pairwise comparisons.

In summary, we present the following contributions:

• We present Chameleon, a family of early-fusion token-based mixed-modal models capable of reasoning over and generating interleaved image-text documents, setting a new bar for open multimodal foundation models.

• We introduce architectural innovations and training techniques that enable the stable and scalable training of early-fusion token-based models, addressing key challenges in mixed-modal learning.

• Through extensive evaluations, we demonstrate state-of-the-art performance across a diverse set of vision-language benchmarks, while maintaining competitive performance on text-only tasks, and high quality image generation, all in the same model.

• We conduct the first large-scale human evaluation on open-ended mixed-modal reasoning and generation, demonstrating the unique capabilities of Chameleon in this new setting.

Chameleon represents a significant step towards realizing the vision of unified foundation models capable of flexibly reasoning over and generating multimodal content.

<span id="page-2-0"></span>
Prompt: I’m bored. Could you show me some cool, quirky-looking birds? Also tell me about them in 2–3 sentences.

![](assets/_page_2_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_2_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:21:15
> 
> ---
> 
> The image is a photograph of a toucan perched on a branch. The bird has a large, colorful beak with a red upper mandible and a yellow lower mandible. Its body is primarily dark blue with some green and yellow accents on its wings. The background is blurred, focusing attention on the toucan.


Response: Absolutely! Here are three birds with some unique features that you might find interesting:

Keel-Billed Toucan The Keel-Billed Toucan is a brightly colored bird native to South American rainforests and woodlands. Its most distinctive feature is its colorful beak, which can grow up to seven inches long and has a distinctive, curved shape. The beak is used for plucking fruit from trees and for social displays. Despite its large size, the Keel-Billed Toucan is a skilled flyer and can often be seen flitting through the treetops. Here is an image of this vibrant bird: <img>

Puffin Puffins are small, aquatic birds that are found in the northern hemisphere. They have distinctive black and white feathers, as well as a brightly colored beak that is used for attracting mates and defending their territory. Puffins are known for their distinctive "puffin" call, which sounds like a loud, guttural chuckle. They feed on fish, which they catch by diving into the water from the air. Puffins are colonial birds, and they breed in large colonies on rocky cliffs and islands. Here is an image of this adorable bird: <img>

![](assets/_page_2_Figure_1.jpg)

> **AI Description:**
> **Source:** `assets/_page_2_Figure_1.jpg`
> 
> **Generated:** 2026-05-16 20:21:10
> 
> ---
> 
> The image is a photograph of a puffin standing on a grassy cliff. The puffin has a white underbelly and black wings with a distinctive red beak and feet. The background shows a clear blue sky and the ocean, indicating a coastal setting. The puffin appears to be in a natural habitat, likely on an island or near the sea.


![](assets/_page_2_Figure_2.jpg)

> **AI Description:**
> **Source:** `assets/_page_2_Figure_2.jpg`
> 
> **Generated:** 2026-05-16 20:21:01
> 
> ---
> 
> The image is a close-up photograph of a bird, specifically a Golden Pheasant. The bird's head is prominently featured, showcasing its vibrant yellow and orange plumage. The feathers have intricate patterns and textures, with the colors blending into each other in a natural and beautiful way. The bird's eye is large and expressive, adding to the overall striking appearance of the image.


Golden Pheasant The Golden Pheasant is a brightly colored bird native to the forests of China. It is a member of the pheasant family and is known for its striking plumage, which features a golden-yellow body, red face, and green tail feathers. The Golden Pheasant is a ground-dwelling bird, and it feeds on a diet of insects, seeds, and fruits. It is a popular game bird, and is often hunted for its meat and feathers. In captivity, the Golden Pheasant is sometimes kept as an ornamental bird, due to its striking appearance. Here is an image of this beautiful bird: <img>

I hope these images and descriptions have helped to brighten your day! If you have any other questions or requests, please don’t hesitate to ask.

Figure 2 Sample interleaved image and text generation from Chameleon. The corresponding images are generated in locations marked by <img>.

<span id="page-3-0"></span>
## 2 Pre-Training

Chameleon represents images, in addition to text, as a series of discrete tokens and takes advantage of the scaling properties of auto-regressive Transformers (Ramesh et al., 2021; Aghajanyan et al., 2022, 2023; Yu et al., 2023). We present any ordering of images and text during training ranging from text-only, to single text/image pairs to full interleaved text-image documents.

![](assets/_page_3_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_3_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:22:01
> 
> ---
> 
> The image contains a stop sign with a noticeable fire or flame effect on its surface, set against a backdrop of dense green foliage. The sign is octagonal with the word "STOP" in white capital letters. The fire effect gives the sign a dramatic and intense appearance, suggesting urgency or danger. The surrounding trees indicate that the sign is located in a wooded or forested area.


Prompt: <img> While chameleons are masters of camouflage in the natural world, let’s see how they might fare in a man-made setting.

1. Analyze the colors, patterns, and textures in this image and identify the most challenging spot for a chameleon to blend in.

2. Then generate an image of a chameleon with the optimal pattern to camouflage itself.

3. Then, rate and explain how successful each attempt is on a scale from 1-10.

Response: The image is of a stop sign in a forested area, with some sunlight filtering through the trees.   
The sign has a red octagonal shape with a white border and the word "STOP" written in white letters.   
In the background, there are green leaves and trees.

Challenging Spot for a Chameleon: The stop sign would be the most challenging spot for a chameleon to blend in due to its bright red color and bold white letters, which would stand out in the forested environment.

Optimal Pattern for Camouflage: To camouflage itself on the stop sign, a chameleon would need to have a pattern that would break up its outline and make it less conspicuous. A pattern with shades

![](assets/_page_3_Figure_1.jpg)

> **AI Description:**
> **Source:** `assets/_page_3_Figure_1.jpg`
> 
> **Generated:** 2026-05-16 20:22:07
> 
> ---
> 
> The image is a photograph of a chameleon perched on a branch. The chameleon has a vibrant orange and green coloration, with its eyes prominently visible and alert. The background appears to be an outdoor setting with some greenery and a blurred structure, possibly a building or fence. The image is credited to AI Research at Meta, as indicated by the text in the bottom left corner.


of brown and red would help the chameleon blend in more effectively. Additionally, a mottled or dappled pattern would help to scatter light and further reduce the chameleon’s visibility.

## <img>

Rating and Explanation: A rating of 2/10 would be given for the chameleon’s ability to blend in on the stop sign. While the chameleon’s green and brown pattern helps to break up its outline, the bright red color of the stop sign will still make it stand out in the forested environment.

Figure 3 Sample generation from Chameleon.

## 2.1 Tokenization

Image Tokenization We train a new image tokenizer based on Gafni et al. (2022), which encodes a 512 × 512 image into 1024 discrete tokens from a codebook of size 8192. For training this tokenizer, we use only licensed images. Given the importance of generating human faces, we up-sample the percentage of images with faces during pre-training by 2 times. A core weakness of our tokenizer is in reconstructing images with a large amount of text, therefore upper bounding the capability of our models, when it comes to heavy OCR-related tasks.

Tokenizer We train a new BPE tokenizer (Sennrich et al., 2016) over a subset of the training data outlined below with a vocabulary size of 65,536, which includes the 8192 image codebook tokens, using the sentencepiece library (Kudo and Richardson, 2018).

<span id="page-4-0"></span>
![](assets/_page_4_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_4_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:21:56
> 
> ---
> 
> The image contains a photograph of a white bowl filled with sliced bananas. In the background, there is a whole banana placed on a textured surface. The image appears to be a simple food presentation, possibly for a recipe or a food blog.


Prompt: What can I bake with this? Give me the full recipe and show me an image with the final dish. <img>

Response: Bananas are a versatile fruit that can be used in various baked goods, such as muffins, bread, cakes, and cookies. They add natural sweetness, moisture, and a subtle banana flavor to the finished product. Additionally, bananas are rich in potassium, fiber, and other nutrients, making them a healthy addition to your baking repertoire.

Here is a simple recipe for banana bread that uses 3 ripe bananas:

## Ingredients:

• 3 ripe bananas, mashed

• 1 1/2 cups all-purpose flour

• 1 tsp baking powder

• 1/2 tsp baking soda

• 1/2 tsp salt

• 1/4 cup unsalted butter, softened

• 1 cup granulated sugar

• 2 large eggs

• 1 tsp vanilla extract

• 1/2 cup chopped walnuts (optional)

## Instructions:

• Preheat your oven to 350°F (180°C). Grease a 9x5-inch loaf pan and set it aside.

• In a medium bowl, whisk together the flour, baking powder, baking soda, and salt. Set aside.

• In a large mixing bowl, use an electric mixer to cream together the butter and sugar until light and fluffy, about 2-3 minutes.

• Beat in the eggs one at a time, followed by the mashed bananas and vanilla extract.

![](assets/_page_4_Figure_1.jpg)

> **AI Description:**
> **Source:** `assets/_page_4_Figure_1.jpg`
> 
> **Generated:** 2026-05-16 20:22:11
> 
> ---
> 
> The image contains a photograph of a sliced loaf of banana bread on a white rectangular plate. The bread appears moist and has a golden-brown crust. The slices are neatly arranged, showcasing the texture and color of the interior. The background is a wooden surface, adding a rustic feel to the presentation.


• Gradually mix in the dry ingredients until just combined, being careful not to overmix.

• If using walnuts, fold them into the batter.

• Pour the batter into the prepared loaf pan and smooth the top.

• Bake for 50-60 minutes, or until a toothpick inserted into the center of the bread comes out clean.

• Remove the bread from the oven and let it cool in the pan for 10 minutes before transferring it to a wire rack to cool completely. <img>

Figure 4 Sample Chameleon outputs.

## 2.2 Pre-Training Data

We delineate the pre-training stage into two separate stages. The first stage takes up the first 80% of training while the second stage takes the last 20%. For all Text-To-Image pairs we rotate so that 50% of the time the image comes before the text (i.e., captioning).

## 2.2.1 First Stage

In the first stage we use a data mixture consisting of the following very large scale completely unsupervised datasets.

Text-Only: We use a variety of textual datasets, including a combination of the pre-training data used to train LLaMa-2 (Touvron et al., 2023) and CodeLLaMa (Roziere et al., 2023) for a total of 2.9 trillion text-only tokens.

<span id="page-5-0"></span>

### Full Page Description (Page 5)

**Source:** `assets/_page_5_Asset_1.jpg`

**Generated:** 2026-05-16 20:21:43

---

The image is a line graph with two lines representing different conditions: "w/o QK-norm" and "w/ QK-norm". The x-axis is labeled "Step" and ranges from 0 to 175k, while the y-axis is labeled "Training Loss" and ranges from 3.4 to 3.7. The graph shows the training loss over the number of steps for two scenarios. The "w/ QK-norm" line consistently has a lower training loss compared to the "w/o QK-norm" line, indicating that the QK-norm condition leads to better performance in terms of training loss. The graph suggests that the QK-norm condition helps in achieving a lower training loss as the number of steps increases.


### Full Page Description (Page 5)

**Source:** `assets/_page_5_Asset_0.jpg`

**Generated:** 2026-05-16 20:21:34

---

The image is a line graph that compares the output norm over training steps for three different conditions: "w/ QK-norm and dropout," "w/o dropout," and "w/o QK-norm or dropout." The x-axis represents the training step, ranging from 0k to 30k, while the y-axis represents the output norm, ranging from 0 to 35. The graph shows that the "w/ QK-norm and dropout" condition keeps the output norm relatively low and stable throughout training, whereas the "w/o QK-norm or dropout" condition shows a significant increase in output norm over time. The "w/o dropout" condition also shows a steady increase but at a slower rate compared to the "w/o QK-norm or dropout" condition.


### Full Page Description (Page 5)

**Source:** `assets/_page_5_Asset_2.jpg`

**Generated:** 2026-05-16 20:22:19

---

The image is a line graph showing the training loss over steps for two different conditions: "w/o dropout" (without dropout) and "w/ dropout" (with dropout). The x-axis represents the step number, ranging from 0k to 80k, while the y-axis represents the training loss, ranging from 3.4 to 3.9. The graph illustrates that the training loss decreases over time for both conditions, but the "w/ dropout" condition consistently has a slightly lower training loss compared to the "w/o dropout" condition. The graph also shows some fluctuations in the training loss for both conditions, with the "w/ dropout" condition experiencing more pronounced fluctuations.

Text-Image: The text-image data for pre-training is a combination of publicly available data sources and licensed data. The images are then resized and center cropped into 512 × 512 images for tokenization. In total, we include 1.4 billion text-image pairs, which produces 1.5 trillion text-image tokens.

Text/Image Interleaved: We procure data from publicly available web sources, not including data from Meta’s products or services, for a total of 400 billion tokens of interleaved text and image data similar to Laurençon et al. (2023). We apply the same filtering for images, as was applied in Text-To-Image.

## 2.2.2 Second Stage

In the second stage, we lower the weight of the first stage data by 50% and mix in higher quality datasets while maintaining a similar proportion of image text tokens.

We additionally include a filtered subset of the train sets from a large collection of instruction tuning sets.

## 2.3 Stability

It was challenging to maintain stable training when scaling the Chameleon models above 8B parameters and 1T tokens, with instabilities often only arising very late in training. We adopted to following recipe for architecture and optimization to achieve stability.

Architecture Our architecture largely follows LLaMa-2 (Touvron et al., 2023). For normalization, we continue to use RMSNorm (Zhang and Sennrich, 2019); we use the SwiGLU (Shazeer, 2020) activation function and rotary positional embeddings (RoPE) (Su et al., 2021).

We found that the standard LLaMa architecture showed complex divergences due to slow norm growth in the mid-to-late stages of training. We narrowed down the cause of the divergence to the softmax operation being problematic when training with multiple modalities of significantly varying entropy due to the translation invariant property of softmax (i.e., sof tmax(z) = sof tmax(z + c)). Because we share all weights of the model across modalities, each modality will try to “compete” with the other by increasing its norms slightly; while not problematic at the beginning of training, it manifests in divergences once we get outside the effective representation range of bf16 (In Figure 6b, we show that ablations without image generation did not diverge). In a unimodal setting, this problem has also been named the logit drift problem (Wortsman et al., 2023). In Figure 5a, we plot the norms of the output of the last transformer layer as training progresses and we find that although training divergences can manifest after as much as even 20-30% of training progress, monitoring uncontrolled growth of output norms is strongly correlated with predicting future loss divergence.

The softmax operation appears in two places in transformers: the core attention mechanism and the softmax over the logits. As inspired by Dehghani et al. (2023) and Wortsman et al. (2023), we first deviate from the Llama architecture by using query-key normalization (QK-Norm). QK-Norm directly controls the norm growth of input to the softmax by applying layer norm to the query and key vectors within the attention.

<span id="page-6-0"></span>

### Full Page Description (Page 6)

**Source:** `assets/_page_6_Asset_2.jpg`

**Generated:** 2026-05-16 20:20:10

---

The image is a line graph showing the training loss over steps during the training of a model. The x-axis represents the step number, ranging from 0 to 10,000, while the y-axis represents the training loss, ranging from 3.5 to 6.0. Two lines are plotted: one labeled "w/o norm reordering" and the other labeled "w/ norm reordering." The "w/o norm reordering" line shows a more jagged and higher training loss trend compared to the "w/ norm reordering" line, which is smoother and shows a lower training loss. The graph indicates that the model with norm reordering achieves a lower training loss compared to the model without norm reordering.


### Full Page Description (Page 6)

**Source:** `assets/_page_6_Asset_0.jpg`

**Generated:** 2026-05-16 20:20:47

---

The image is a line graph showing the training loss over steps for two different model sizes, 7b and 34b. The x-axis represents the number of steps, ranging from 0 to 600k, while the y-axis represents the training loss, ranging from approximately 2.8 to 3.2. The graph includes two lines: one for the 7b model (yellow) and another for the 34b model (brown). The 34b model consistently has a lower training loss throughout the steps, indicating better performance. The 7b model shows a more gradual decrease in training loss.


### Full Page Description (Page 6)

**Source:** `assets/_page_6_Asset_1.jpg`

**Generated:** 2026-05-16 20:20:39

---

The image is a line graph showing the training loss over a number of steps during the training of a 7B model without image generation. The x-axis represents the number of steps, ranging from 0 to 250k, while the y-axis represents the training loss, ranging from 0.95 to 1.15. The graph indicates a general downward trend in training loss as the number of steps increases, suggesting that the model is improving over time. The line is labeled "7B w/o image generation," indicating that this is a comparison to a similar model with image generation capabilities.

In Figure 5b, we show training loss curves for Chameleon-7B with and without QK-Norm, and the latter diverges after approximately 20% of a training epoch.

We found that to stabilize Chameleon-7B by controlling norm growth, it was necessary to introduce dropout after the attention and feed-forward layers, in addition to QK-norm (see Figure 5c). However, this recipe was not enough to stabilitize, Chameleon-34B, which required an additional re-ordering of the norms. Specifically, we use the strategy of normalization proposed in Liu et al. (2021), within the transformer block. The benefit of the Swin transformer normalization strategy is that it bounds the norm growth of the feedforward block, which can become additionally problematic given the multiplicate nature of the SwiGLU activation function. If h represents the hidden vector at time-step t after self-attention is applied to input x,

$$
{ \mathsf { C h a m e l e o n } } { \mathsf { - } } 3 4 { \mathsf { B } } { \mathsf { : } } \quad h = x + { \mathrm { a t t e n t i o n } } \_ { \mathrm { n o r m } } ( { \mathrm { a t t e n t i o n } } ( x ) )
$$

$$
\begin{array} { l l } { { } } & { { \mathrm { o u t p u t } = h + \mathrm { f f n \_ n o r m } ( \mathrm { f e e d \_ f o r w a r d } ( h ) ) } } \\ { { \mathsf { L l a m a 2 } \colon } } & { { h = x + \mathrm { a t t e n t i o n } ( \mathrm { a t t e n t i o n \_ n o r m } ( x ) ) } } \\ { { } } & { { \mathrm { o u t p u t } = h + \mathrm { f e e d \_ f o r w a r d } ( \mathrm { f i n \_ n o r m } ( h ) ) } } \end{array}
$$

There was no difference in perplexity when training a model from scratch with and without the normalization re-ordering until the divergence of the LLaMa-2 parameterization. Additionally, we found that this type of normalization did not work well in combination with dropout and therefore, we train Chameleon-34B without dropout (Figure 6c). Furthermore, we retroactively found that Chameleon-7B can also be stably trained without dropout, when using norm-reordering, but QK-norm is essential in both cases. We plot training curves for the first 600k steps for both Chameleon-7B and Chameleon-34B in Figure 6a.

Optimization Our training process uses the AdamW optimizer (Loshchilov and Hutter, 2017), with $\beta _ { 1 }$ set to 0.9 and $\beta _ { 2 }$ to 0.95, with an $\epsilon = 1 0 ^ { - 5 }$ We use a linear warm-up of 4000 steps with an exponential decay schedule of the learning rate to 0. Additionally, we apply a weight decay of 0.1 and global gradient clipping at a threshold of 1.0. We use a dropout of 0.1 (Srivastava et al., 2014) for Chameleon-7B for training stability, but not for Chameleon-34B (see Figure 5c and 6c).

The application of QK-Norm while helping the inner softmaxes within the Transformer does not solve the problem of logit shift in the final softmax. Following Chowdhery et al. (2022); Wortsman et al. (2023), we apply z-loss regularization. Specifically, we regularize the partition function Z of the softmax function $\sigma ( x ) _ { i } = \overset { e ^ { x _ { i } } } Z$ where $\textstyle Z = \sum _ { i } e ^ { x _ { i } }$ by adding $\mathrm { i } 0 ^ { - 5 } \log ^ { 2 } Z$ to our loss function.

For Chameleon-7B it was important to use both dropout and z-loss to achieve stability, while Chameleon-34B only required z-loss (Figure 6c).

Chameleon-7B was trained with a global batch size of $2 ^ { 2 3 } ~ ( \sim 8 \mathrm { M } )$ tokens and Chameleon-34B was trained with a global batch size of $3 \times 2 ^ { 2 2 } ~ ( \sim \mathrm { { 1 2 M } ) }$ tokens. We do 2.1 epochs over our full training dataset for a total of 9.2 trillion tokens seen during training. We show the first 600k steps of training (55% for Chameleon-7B and 80% for Chameleon-34B) in Figure 6a.

<span id="page-7-0"></span>
Table 1 Summary of core architecture and optimization decisions made in Chameleon in contrast to LLaMa-1 and LLaMa-2.
![](tables/table_pg7_num0.csv)

Pre-Training Hardware Our model pretraining was conducted on Meta’s Research Super Cluster (RSC) (Lee and Sengupta, 2022), and our alignment was done on other internal research clusters. NVIDIA A100 80 GB GPUs power both environments. The primary distinction is the interconnect technology: RSC employs NVIDIA Quantum InfiniBand, whereas our research cluster utilizes Elastic Fabric. We report our GPU usage for pre-training in Table 2.

Table 2 Chameleon Model Pre-Training Resource Usage
![](tables/table_pg7_num1.csv)

## 2.4 Inference

To support alignment and evaluation, both automated and human, and to demonstrate the applicationreadiness of our approach, we augment the inference strategy with respect to interleaved generation to improve throughput and reduce latency.

Autoregressive, mixed-modal generation introduces unique performance-related challenges at inference time. These include:

• Data-dependencies per-step — given that our decoding formulation changes depending on whether the model is generating images or text at a particular step, tokens must be inspected at each step (i.e. copied from the GPU to the CPU in a blocking fashion) to guide control flow.

• Masking for modality-constrained generation — to facilitate exclusive generation for a particular modality (e.g. image-only generation), tokens that do not fall in a particular modality space must be masked and ignored when de-tokenizing.

• Fixed-sized text units — unlike text-only generation, which is inherently variable-length, token-based image generation produces fixed-size blocks of tokens corresponding to an image.

Given these unique challenges, we built a standalone inference pipeline based on PyTorch (Paszke et al., 2019) supported with GPU kernels from xformers (Lefaudeux et al., 2022).

Our inference implementation supports streaming for both text and images. When generating in a streaming fashion, token-dependent conditional logic is needed at each generation step. Without streaming, however, blocks of image tokens can be generated in a fused fashion without conditional computation. In all cases, token masking removes branching on the GPU. Even in the non-streaming setting, however, while generating text, each output token must be inspected for image-start tokens to condition image-specific decoding augmentations.

<span id="page-8-0"></span>
Table 3 Supervised Fine-Tuning Dataset Statistics
![](tables/table_pg8_num0.csv)

## 3 Alignment

We follow recent work in using a light weight alignment stage based on supervised fine tuning on carefully curated high quality datasets (Zhou et al., 2023). We include a range of different types of data, targeting both exposing model capabilities and improving safety.

## 3.1 Data

We separate our supervised fine-tuning (SFT) dataset into the following categories: Text, Code, Visual Chat, Image Generation, Interleaved Text/Image Generation, and Safety. We include examples from each category from the Chameleon-SFT dataset in Figure 7.

We inherit the Text SFT dataset from LLaMa-2 (Touvron et al., 2023) and the Code SFT from CodeLLaMa (Roziere et al., 2023). For the Image Generation SFT dataset, we curate highly aesthetic images by applying and filtering each image in our licensed data, with an aesthetic classifier from Schuhmann et al. (2022). We first select images rated as at least six from the aesthetic classifier and then select the top 64K images closest in size and aspect ratio to 512 × 512 (the native resolution of our image tokenizer).

For both Visual Chat and Interleaved Text/Image Generation SFT data, we focused on very high-quality data collection using third-party vendors following a similar strategy recommended by Touvron et al. (2023); Zhou et al. (2023). We do not include any Meta user data. We present our dataset’s statistics in Table 3.

Safety Data We include a collection of prompts that can potentially provoke the model to produce unsafe content, and match them with a refusal response (e.g. “I can’t help with that.”). These prompts cover a wide variety of sensitive topics, such as violence, controlled substances, privacy, and sexual content. Our collection of safety tuning data includes examples from LLaMa-2-Chat (Touvron et al., 2023), synthetic text-based examples generated with Rainbow Teaming (Samvelyan et al., 2024), image generation prompts manually selected from Pick-A-Pic (Kirstain et al., 2023) for safety testing, examples for cyber security safety (Roziere et al., 2023), as well as mixed-modal prompts collected internally through manual annotation and automatic expansion (Honovich et al., 2022). Collecting mixed-modal prompts is of particular importance, since it addresses potential multi-modal attack vectors, which are outside the distribution of text-only and text-to-image safety tuning datasets.

## 3.2 Fine-Tuning Strategy

Data Balancing We found that balancing modalities within the SFT stage is important for high quality alignment. Specifically during the SFT stage, if there is a severe imbalance between pairings of modalities (or when a specific modality should trigger), the model learns an unconditional prior of generating that modality which can either mute or over exaggerate the generation of a single modality.

Optimization Our supervised fine-tuning strategy incorporates a cosine learning rate schedule, starting at an initial rate of 1e-5, combined with a weight decay set at 0.1. We maintain a batch size of 128, accommodating sequences up to 4096 tokens. During fine-tuning, each dataset instance comprises a paired prompt and its corresponding answer. To improve efficiency, we pack as many prompts and answers as possible into each

<span id="page-9-0"></span>
Prompt: Can you write a poem using rhyming couplets?   
Response: Sure, here’s a short poem using rhyming couplets:

The sun sets in the sky, A beautiful sight to the eye. The stars come out to play, A magical sight to behold.

The wind whispers through the trees, A gentle breeze that brings ease. The birds sing their sweet melodies, A soothing sound that calms the soul. The world is full of wonder, A place of beauty and thunder. So let us cherish every moment, And bask in the beauty all around us.

(a) Text

Prompt: Given a list of integers,   
find the first number that ap  
pears at least twice.   
Response:   
def find\_duplicate(my\_list): seen = set() for num in my\_list: if num in seen: return num seen.add(num) return None

(b) Code

Prompt: Renaissance square of   
Stepanakert   
Response:

(c) Image Generation

![](assets/_page_9_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_9_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:20:26
> 
> ---
> 
> The image is a photograph of a large, open square with a prominent church or cathedral featuring a pointed roof in the center. The architecture suggests a European or Mediterranean style. The sky is filled with dramatic, dark clouds, creating a moody atmosphere. The square is bordered by buildings of various sizes and styles, and there are a few people walking on the paved ground. The overall scene conveys a sense of grandeur and historical significance.


![](assets/_page_9_Figure_1.jpg)

> **AI Description:**
> **Source:** `assets/_page_9_Figure_1.jpg`
> 
> **Generated:** 2026-05-16 20:19:51
> 
> ---
> 
> The image is a photograph of a decorative vase containing a bouquet of flowers. The vase is green with a textured pattern and sits on a surface outdoors, surrounded by autumn leaves. The flowers include a purple pansy with yellow center and red leaves, adding a vibrant touch to the scene. The background shows a blurred outdoor setting with greenery, suggesting a garden or park environment.


Prompt: Which elements contribute to this flower arrangement’s distinctiveness, and what sets its surroundings apart?   
Response: The image features an autumnal-themed floral arrangement in a classically designed vase. The arrangement stands out with its striking combination of red maple leaves, purple flowers, and pink-red roses. The vase’s rosettes give it an antique design. In the background, blurry green vegetation is visible, creating a contrast to the autumnal theme of the bouquet. The red maple leaves, both in the arrangement and scattered around the vase, evoke autumn, while the green vegetation suggests a more summer-like setting. The outdoor setting is bright and sunny, adding to the contrast between the arrangement and its settings.

(d) Visual Chat

Figure 7 Example alignment data for different categories.

Gyumri is an urban municipal community and the second-largest city in Armenia, serving as the administrative center of Shirak Province in the northwestern part of the country.

![](assets/_page_9_Figure_2.jpg)

> **AI Description:**
> **Source:** `assets/_page_9_Figure_2.jpg`
> 
> **Generated:** 2026-05-16 20:20:32
> 
> ---
> 
> The image is a photograph of a cobblestone street in an urban setting. The street is lined with traditional-style buildings on both sides, featuring a mix of architectural styles. The buildings have a warm, earthy tone with red and brown hues. The sky above is partly cloudy, with patches of blue visible, suggesting a pleasant day. The street appears to be quiet, with no visible pedestrians or vehicles, giving the scene a calm and serene atmosphere.


Archaeological excavations conducted throughout the Soviet period have shown that the area of modern-day Gyumri has been populated since at least the third millennium BC.

(e) Interleaved Text/Image Generation
![](assets/_page_9_Figure_3.jpg)

> **AI Description:**
> **Source:** `assets/_page_9_Figure_3.jpg`
> 
> **Generated:** 2026-05-16 20:20:52
> 
> ---
> 
> The image depicts a series of stone walls arranged in a grid-like pattern, likely forming the remains of an ancient structure or enclosure. The walls are constructed from large, uneven stones, and the ground is covered with dry grass and patches of dirt. The background shows a hilly landscape with sparse vegetation and a partly cloudy sky. The image suggests a historical or archaeological site, possibly used for agricultural or ceremonial purposes.


sequence, inserting a distinct token to delineate the end of a prompt and the beginning of an answer. We use an autoregressive training objective, selectively masking the loss for the prompt tokens. This targeted approach allows us to optimize the model exclusively based on the answer tokens, which provides slight gains overall. We also apply a dropout of 0.05. Additionally, we maintain the same zloss that was used during

<span id="page-10-0"></span>
## Advice: 10.2%

What does a meningitis rash look like? What are the other symptoms I should be on the lookout for?

## How-to: 12.5%

How do I properly clean my TV screen? I used Windex and now there are towel fibers and wipe marks all over. Show me some reference photos.

## Explanation: 14.4%

I've been studying classical French art, and my favorite so far is his painting seen here : <img> Could you please give me a few images of other contemporary artworks that have this same aesthetic?

![](assets/_page_10_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:21:25
> 
> ---
> 
> The image contains a painting with a Baroque style, featuring a fantastical scene with elaborate floral and architectural elements. The colors are rich and vibrant, with a dominant use of gold and blue hues. The composition is dynamic, with a sense of movement and depth, suggesting a celestial or mythological setting. The painting is likely intended to evoke a sense of grandeur and opulence, typical of the Baroque period.


## Hypothetical: 5.6%

What would the modern-day vehicle look like if oil had never been discovered?

## Brainstorming: 18.6%

Show me a Middle Eastern alternative to these dishes. <img1> <img2>

![](assets/_page_10_Figure_1.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_1.jpg`
> 
> **Generated:** 2026-05-16 20:21:48
> 
> ---
> 
> The image contains a photograph of a layered dessert. The dessert appears to be a type of mille-feuille or similar layered pastry, with multiple thin layers of golden-brown puff pastry. Between the layers, there is a creamy filling that is swirled with what looks like chocolate or a similar dark-colored sauce. The top layer is dusted with powdered sugar, giving it a delicate appearance. The dessert is presented on a dark surface, possibly a plate or a cutting board.


![](assets/_page_10_Figure_2.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_2.jpg`
> 
> **Generated:** 2026-05-16 20:22:41
> 
> ---
> 
> The image is a photograph of a slice of panettone, a traditional Italian sweet bread. The panettone is golden brown with visible raisins and other dried fruits embedded within the dough. The slice is placed on a white plate, and the background includes a red cloth, possibly a tablecloth, adding a warm tone to the image. The overall presentation suggests a festive or holiday context, as panettone is often associated with Christmas in Italy.


## Article: 3.1%

Write me an introduction to a story about knick-knacks, and finish the story by shifting the focus with an image.

## Story: 3.9%

Can you create and illustrate a short story for children about an octopus that can't stop eating pizza?

## Identification: 9.3 %

![](assets/_page_10_Figure_3.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_3.jpg`
> 
> **Generated:** 2026-05-16 20:21:53
> 
> ---
> 
> The image is a photograph showing a person kneeling beside a horse in what appears to be a stable or arena setting. The person is wearing a dark outfit and has long hair. The horse is light brown with a darker mane and tail. The background includes other horses and people, suggesting this might be a horse show or competition. There is a number tag on the horse's halter, but the number is not clearly visible.


Is the below image a Shetland Pony? If not, what is it, and can you show me a Shetland Pony? <img>

## Comparison: 9.6%

Please tell me what the difference between these two creatures is, and show me some more examples. <img1> <img2>

![](assets/_page_10_Figure_4.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_4.jpg`
> 
> **Generated:** 2026-05-16 20:21:20
> 
> ---
> 
> The image is a photograph of a green tree frog perched on a leaf. The frog is positioned on the left side of the image, with its body facing towards the right. The leaf it is resting on appears to be part of a palm frond, with visible veins and a textured surface. The background is blurred, focusing attention on the frog and the leaf. The frog's skin has a smooth texture with visible ridges, and its eyes are open, giving it a alert appearance.


![](assets/_page_10_Figure_5.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_5.jpg`
> 
> **Generated:** 2026-05-16 20:21:06
> 
> ---
> 
> The image is a photograph of a toad. The toad is positioned centrally against a plain white background. It has a dark brown coloration with visible texture and patterns on its skin. The toad's eyes are open, and its mouth is slightly open. The image is clear and well-lit, allowing for a detailed view of the toad's features.


## Report: 5.4%

Who designed the church in and what's the name of the Church? <img> Can you please provide me with additional photos of famous landmarks designed by the same architect?

![](assets/_page_10_Figure_6.jpg)

> **AI Description:**
> **Source:** `assets/_page_10_Figure_6.jpg`
> 
> **Generated:** 2026-05-16 20:20:57
> 
> ---
> 
> The image is a photograph of a large, ornate cathedral with intricate architectural details. The cathedral is under construction, as evidenced by the cranes and scaffolding visible on the structure. The surrounding area includes a well-maintained park with greenery and pathways, and there are buildings in the background, indicating an urban setting. The cathedral is the focal point of the image, showcasing its grand scale and architectural complexity.


## Other: 5.2%

Create a decal for my truck that features running horses as well as the TRD insignia. Use black to gray gradients.

## Reasoning: 2.1%

What is typically found at a construction site? Show me a construction site that has a crane.

Figure 8 Task categories and examples of prompts. Image attributions: Seguin (2010); Agriflanders (2009); Tuszyński (2015); Sokolov (2022).

pre-training. During supervised fine-tuning, images in the prompt are resized with border padding to ensure that all the information is available in the image, whereas images in the answer are center-cropped to ensure visually good image generation quality.

## 4 Human Evaluations and Safety Testing

Chameleon has significant new mixed modal understanding and generation abilities that cannot be measured with existing benchmarks. In this section, we detail how we conduct human evaluations on large multi-modal language models’ responses to a set of diverse prompts that regular users may ask daily. We first introduce how we collect the prompts and then describe our baselines and evaluation methods, along with the evaluation results and analysis. A safety study is also included in this section.

## 4.1 Prompts for Evaluation

We work with a third-party crowdsourcing vendor to collect a set of diverse and natural prompts from human annotators. Specifically, we ask annotators to creatively think about what they want a multi-modal model to generate for different real-life scenarios. For example, for the scenario of “imagine you are in a kitchen”, annotators may come up with prompts like “How to cook pasta?” or “How should I design the layout of my island? Show me some examples.” The prompts can be text-only or text with some images, and the expected responses should be mixed-modal, containing both text and images.

After collecting an initial set of prompts, we ask three random annotators to evaluate whether the prompts are clear and whether they expect the responses to contain images. We use a majority vote to filter unclear prompts and prompts that don’t expect mixed-modal responses. In the end, our final evaluation set contains 1,048 prompts: 441 (42.1%) are mixed-modal (i.e., containing both text and images), and the remaining 607 (57.9%) are text-only.

To better understand the tasks users would like a multi-modal AI system to fulfill, we manually examine the prompts and classify them into 12 categories. The description of these task categories1, as well as their example prompts, can be found in Figure 8.

<span id="page-11-0"></span>

### Full Page Description (Page 11)

**Source:** `assets/_page_11_Asset_0.jpg`

**Generated:** 2026-05-16 20:20:01

---

The image is a bar chart showing the percentage of task fulfillment rates for different models: Chameleon, Gemini+, GPT-4V+, Gemini, and GPT-4V. The x-axis represents the task fulfillment categories: "Fulfills," "Partially fulfills," and "Does not fulfill." The y-axis represents the percentage of fulfillment. The chart highlights that the "Partially fulfills" category has the highest percentage for all models, with Gemini+ and GPT-4V+ showing the highest percentages. The "Fulfills" category has the lowest percentage for all models, with Chameleon having the highest percentage. The "Does not fulfill" category has the lowest percentage for all models, with Chameleon having the highest percentage.


### Full Page Description (Page 11)

**Source:** `assets/_page_11_Asset_1.jpg`

**Generated:** 2026-05-16 20:20:21

---

The image is a horizontal stacked bar chart with four categories: Gemini+, GPT-4V+, Gemini, and GPT-4V. Each category is further divided into three segments representing Wins, Ties, and Loses, with corresponding percentages. The chart visually compares the performance of these categories across the three outcomes. Gemini+ has the highest percentage of Wins at 41.5%, while Gemini has the highest percentage of Wins at 53.5%. The percentages for Ties and Loses are also displayed, with Gemini+ having the highest percentage of Ties at 34.5%. The chart uses a color-coded legend to differentiate between Wins (blue), Ties (orange), and Loses (green).

## 4.2 Baselines and Evaluations

We compare Chameleon 34B with OpenAI GPT-4V and Google Gemini Pro by calling their APIs. While these models can take mixed-modal prompts as input, their responses are text-only. We create additional baselines by augmenting GPT-4V and Gemini responses with images to have even stronger baselines. Specifically, we instruct these models to generate image captions by adding the following sentence at the end of each original input prompt: “If the question requires an image to be generated, then generate an image caption instead and enclose the caption in a pair of ⟨caption⟩ ⟨/caption⟩ tags.” We then use OpenAI DALL-E 3 to generate images conditioned on these captions and replace the captions in the original responses with those generated images. We denote the enhanced responses as GPT-4V+ and Gemini+ in this section. Working with the same third-party crowdsourcing vendor, we conduct two types of evaluations to measure the model performance: absolute and relative.

## 4.2.1 Absolute Evaluation

For absolute evaluations, the output of each model is judged separately by asking three different annotators a set of questions regarding the relevance and quality of the responses. Below, we give detailed results and analysis on the most critical question, whether the response fulfills the task described in the prompt.

On task fulfillment, we ask annotators whether the response fulfills, partially fulfills, or does not fulfill the task described in the prompt. As shown in Figure 9a, much more of Chameleon’s responses are considered to have completely fulfilled the tasks: 55.2% for Chameleon vs. 37.6% of Gemini+ and 44.7% of GPT-4V+. When judging the original responses of Gemini and GPT-4V, the annotators consider much fewer prompts to be fully fulfilled: Gemini completely fulfills 17.6% of the tasks and GPT-4V 23.1%. We suspect that because all the prompts expect mixed-modal output, the text-only responses from Gemini and GPT-4V might be viewed as only partially completing the tasks by the annotators.

The task fulfillment rates in each category and in each input modality can be found in Appendix B. The task categories that Chameleon performs well include Brainstorming, Comparison, and Hypothetical, and the categories Chameleon needs to improve include Identification and Reasoning. On the other hand, we don’t see that the model performance differs a lot when comparing mixed-modality and text-only prompts, although Chameleon seems to perform slightly better on text-only prompts, while Gemini+ and GPT-4V+ are slightly better on mixed-modal ones. Figure 2 shows an example of Chameleon’s response to a brainstorming prompt.

<span id="page-12-0"></span>

### Full Page Description (Page 12)

**Source:** `assets/_page_12_Asset_0.jpg`

**Generated:** 2026-05-16 20:22:37

---

The image is a horizontal bar chart with three categories of agreement: "All," "Two," and "None." The chart displays counts for various factors related to image and language quality, including "Containing images," "Image quality," "Image relevance," "Language quality," "Objectionable content," "Relevance," "Task fulfillment," and "Accuracy." The "Objectionable content" factor has the highest count, followed by "Language quality," and then "Task fulfillment." The "Image quality" and "Image relevance" factors have the lowest counts. The chart uses blue bars for "All," orange bars for "Two," and green bars for "None."

## 4.2.2 Relative Evaluation

For relative evaluations, we directly compare Chameleon with each baseline model by presenting their responses to the same prompt in random order and asking human annotators which response they prefer. The options include the first response, the second response, and about the same. Figure 9b shows Chameleon’s win rates2 over the baselines. Compared with Gemini+, Chameleon’s responses are better in 41.5% of the cases, 34.5% are tie, and 24.0% are inferior. Annotators also think that Chameleon’s responses are slightly more often better than GPT-4V+, with 35.8% win, 31.6% tie, and 32.6% loss. Overall, Chameleon has win rates of 60.4% and 51.6% over Gemini+ and GPT-4V+, respectively. When compared with the original responses from Gemini without the augmented images, Chameleon’s responses are considered better in 53.5% of the cases, 31.2% are tied, and 15.3% are inferior. Chameleon’s responses are also considered better than GPT-4V more frequently, with 46.0% win, 31.4% tie, and 22.6% loss. Chameleon’s win rates over Gemini and GPT-4V are 69.1% and 61.7%, respectively.

## 4.3 Inter-annotator Agreement

Every question in our evaluation is answered by three different human annotators, and we take the majority votes as the final answer. To understand the quality of the human annotators and whether the questions we asked are reasonably designed, we examine the level of agreement between different annotators.

The levels of agreement on each question in the absolute evaluation are shown in Figure 10.

For questions about simple, objective properties of the responses, we very rarely see three annotators disagree with each other. For example, annotators have unanimous judgments on whether the model responses contain objectionable content (e.g., hate speech); in this case, all models produce safe responses. For some questions, such as whether the response fulfills the task or whether the model interprets the prompt correctly, when one annotator’s judgment differs from the other two’s, the decision is usually still close (e.g., fulfills vs. partially fulfills) rather than opposite (e.g., fulfills vs. does not fulfill ).3

<span id="page-13-0"></span>
Table 4 The inter-annotator agreement on relative evaluations.
![](tables/table_pg13_num0.csv)

For the relative evaluation, Table 4 shows the numbers of cases where all three annotators agree, two annotators agree, and there is no agreement. For each model pair, we have a bit higher than 10% of the cases where there is no agreement among the three annotators (considered as a tie in our evaluation.) On about 28% to 35% of the pairs, all annotators have unanimous judgments, and in about 55% to 60% of the pairs, one annotator differs from other two. This may be interpreted as Chameleon performing similarly to other baselines in many cases, making the relative evaluation challenging.4

## 4.4 Safety Testing

We crowdsource prompts that provoke the model to create unsafe content in predefined categories such as self-harm, violence and hate, and criminal planning. These prompts cover both text and mixed-modal inputs, as well as intents to produce unsafe text, images, or mixed-modal outputs. We generate the model’s response to each prompt, and ask annotators to label whether the response is safe or unsafe with respect to each category’s definition of safety; an unsure option is also provided for borderline responses. Table 5 shows that an overwhelming majority of Chameleon’s responses are considered safe, with only 78 (0.39%) unsafe responses for the 7B model and 19 (0.095%) for the 30B model.

We also evaluate the model’s ability to withstand adversarial prompting in an interactive session. For that purpose, an internal red team probed the 30B model over 445 prompt-response interactions, including multi-turn interactions. Table 5 shows that of those responses, 7 (1.6%) were considered unsafe and 20 (4.5%) were labeled as unsure. While further safety tuning using RLHF/RLAIF has been shown to further harden the model against jailbreaking and intentional malicious attacks, these results demonstrate that our current safety tuning approach provides significant protection for reasonable, benign usage of this research artifact.

## 4.5 Discussion

Compared to Gemini and GPT-4V, Chameleon is very competitive when handling prompts that expect interleaving, mixed-modal responses. The images generated by Chameleon are usually relevant to the context, making the documents with interleaving text and images very appealing to users. However, readers should be aware of the limitations of human evaluation. First, the prompts used in the evaluation came from crowdsourcing instead of real users who interact with a model. While we certainly have a diverse set of prompts, the coverage may still be limited, given the size of the dataset. Second, partially because our prompts focus on the mixed-modal output, certain visual understanding tasks, such as OCR or Infographics (i.e., interpreting a given chart or plot), are naturally excluded from our evaluation. Finally, at this moment, the APIs of existing multi-modal LLMs provide only textual responses. While we strengthen the baselines by augmenting their output with separately generated images, it is still preferred if we can compare Chameleon to other native mixed-modal models.

<span id="page-14-0"></span>
Table 5 Safety testing on 20,000 crowdsourced prompts and 445 red team interactions provoking the model to produce unsafe content.
![](tables/table_pg14_num0.csv)

## 5 Benchmark Evaluations

Given the general capabilities of Chameleon, there is not a single model that we can directly evaluate against;   
therefore, we evaluate against the best models in every category within our capabilities.

## 5.1 Text

We evaluate the general text-only capabilities of our pre-trained (not SFT’d) model against other state-of-theart text-only large language models. We follow the evaluation protocol outlined by Touvron et al. (2023). Specifically we evaluate all models, using an in-house evaluation platform on the areas of commonsense reasoning, reading comprehension, math problems, and world knowledge. We report our results in Table 6.

Table 6 Comparison of overall performance on collective academic benchmarks against open-source foundational models. ∗ Evaluated using our framework/using API. For GSM8k/MATH, we report maj@1 unless mentioned otherwise. ∗∗ From Gemini et al. (2023).
![](tables/table_pg14_num1.csv)

• Commonsense Reasoning and Reading Comprehension: We report 0-shot performance on the following benchmarks that measure commonsense reasoning and reading comprehension capabilities: PIQA (Bisk et al., 2020), SIQA (Sap et al., 2019), HellaSwag (Zellers et al., 2019), WinoGrande (Sakaguchi et al., 2021), ARC-Easy (Clark et al., 2018), ARC-Challenge (Clark et al., 2018), OpenBookQA (Mihaylov et al., 2018), and BoolQ (Clark et al., 2019). We score the prompt with each candidate answer and compute accuracy using the candidate with the highest score. All baseline model performances except a few are taken directly from the reported sources. We observe that Chameleon-7B and Chameleon-34B are competitive with the corresponding Llama-2 models, with Chameleon-34B even outperforming Llama-2 70B on 5/8 tasks and performing on par with Mixtral 8x7B.

<span id="page-15-0"></span>
• MATH and World Knowledge We report 8-shot performance on GSM8K (Cobbe et al., 2021) i.e., grade school math word problems and 4-shot performance on the MATH (Hendrycks et al., 2021) benchmark. We report maj@N exact match accuracy for both benchmarks by sampling N generations from the model (greedy sampling for N=1) and choosing the answer via majority voting. Despite training for additional modalities, both Chameleon models demonstrate strong math capabilities. On GSM8k, Chameleon-7B outperforms the corresponding Llama-2 models, with performance comparable to Mistral 7B (50.9 vs 52.1 maj@8). Furthermore, Chameleon-34B can outperform Llama2-70B on maj@1 (61.4 vs 56.8) and Mixtral 8x7B on maj@32 (77.0 vs 75.1). Similarly, on MATH, Chameleon-7B outperforms Llama-2 and matches Mistral 7B on maj@4, while Chameleon-34B outperforms Llama2-70B, approaching the performance of Mixtral 8x7B on maj@4 (24.7 vs 28.4).

We also report performance on MMLU (Hendrycks et al., 2020), which measures world/in-domain knowledge and problem-solving abilities using 57 subjects, including elementary mathematics, US history, computer science, and law. Both Chameleon models outperform their Llama-2 counterparts with Chameleon-34B approaching the performance of Mixtral 8x7B/Gemini-Pro (65.8 vs 70.6/71.8).

Overall, Chameleon outperforms LLaMa-2 across the board, with performance approaching Mistral 7B/Mixtral 8x7B (Jiang et al., 2023, 2024) on some tasks. These gains are likely due to multiple factors. First, we do two epochs over the LLaMa-2 pre-training data, and in general use more compute for pretraining. Second, including code data significantly improves performance on text-only reasoning tasks. Lastly, having higher quality data in the last 20% of pre-training significantly improves performance.

## 5.2 Image-To-Text

We next evaluate Chameleon on the segment of tasks that requires text generation conditioned on an image, specifically on image captioning and visual question-answering tasks, and present results of Chameleon-34B in Table 7. Together with our pre-trained model, we also present results with a model fine-tuned on all tasks together (Chameleon-34B-MultiTask), as well as models exclusively fine-tuned for the specific evaluation tasks (Chameleon-34B-SFT).

We evaluate against available open-source late-fusion models: specifically Flamingo 80B (Alayrac et al., 2022), IDEFICS 80B (Laurençon et al., 2023), and Llava-1.5 (Liu et al., 2023a), as well as recent closed-source models, such as Gemini (Gemini et al., 2023) and GPT4-V (OpenAI, 2023). We note that we did not take any special care when formatting the pre-training data to ensure that 0-shot inference can be effectively done. Therefore, we augment the input images or questions with the published prompts used by other models. This was purposefully done to maintain the fidelity of the pre-training data.

• Image Captioning: For image captioning evaluations we report CiDER (Vedantam et al., 2015) scores on the Karpathy test split of MS-COCO (Lin et al., 2014), and the Karpathy test split of Flickr30k (Plummer et al., 2015) using the pycocoevalcap (Chen et al., 2020) package. For Chameleon models, we restrict captions to 30 tokens. We evaluated GPT-4V and Gemini models using several prompts and generation lengths via their APIs and report the best performance that we were able to achieve.

In the open-source pre-trained category, Chameleon-34B (2-shot) outperforms the larger 80B models of both Flamingo and IDEFICS on COCO with 32-shots, while matching their performance on Flickr30k. With respect to fine-tuned/closed-source models, both multi-task and SFT variants of Chameleon-34B outperform all other models on COCO, while for Flickr30k, the SFT model outperforms other models with the multitask model being a close competitor.

• Visual Question Answering: For visual question answering (VQA) we report performance on the testdev split of VQA-v2 (Goyal et al., 2017). For VQA-v2, the pre-trained Chameleon-34B model with 2-shots matches the 32-shot performance of the larger Flamingo and IDEFICS models, while for finetuned/closed models, Chameleon-34B-Multitask approaches the performance of IDEFICS-80B-Instruct and Gemini Pro, but trails larger models such as Flamingo-80B-FT, GPT-4V, and Gemini Ultra. Llava-1.5 outperforms Chameleon-34B on VQAv2 potentially owing to its additional fine-tuning on conversations from GPT-4, ShareGPT (ShareGPT, 2023), GQA (Hudson and Manning, 2019), and region-level VQA datasets, but significantly trails behind on the other tasks.

<span id="page-16-0"></span>
Table 7 Model Performances on Image-to-Text Capabilities. ∗ Evaluated using API.
![](tables/table_pg16_num0.csv)

In general, we find Chameleon is fairly competitive on both image captioning and VQA tasks. It rivals other models by using much fewer in-context training examples and with smaller model sizes, in both pre-trained and fine-tuned model evaluations.

## 6 Related Work

Chameleon builds upon the lineage of works exploring token-based approaches for multimodal learning. The idea of using discrete tokens to represent continuous modalities like images was first explored in works like BEiT (Bao et al., 2021), which proposed a self-supervised vision representation learning method based on tokenized image patches. Aghajanyan et al. (2022) extended this idea to learning from mixed-modal documents through interleaved image and text tokens, allowing for joint reasoning over both modalities within a unified architecture. CM3Leon (Yu et al., 2023) further scaled up this approach to autoregressive text-to-image generation, building on the initial proposal of token-based image generation in DALL-E (Ramesh et al., 2021).

As a fully token-based early-fusion model, Chameleon differs from late-fusion approaches like Flamingo (Alayrac et al., 2022) which encode images and text separately before combining them at a later stage. Other models like LLaVA (Liu et al., 2023a), IDEFICS (Laurençon et al., 2023), and VisualGPT (Chen et al., 2022) also maintain separate image and text encoders. In contrast, Chameleon’s unified token space allows it to seamlessly reason over and generate interleaved image and text sequences, without the need for modality-specific components. This early-fusion approach, however, comes with significant challenges in terms of representation learning and alignment, as discussed in Baltrušaitis et al. (2018).

The most similar model to Chameleon is Gemini (Gemini et al., 2023), which also uses an early-fusion token-based approach. However, a key difference is that Gemini uses separate image decoders, whereas Chameleon is an end-to-end dense model without any routing components. This makes Chameleon a more general-purpose model for both multimodal understanding and generation tasks, similar in spirit to the Perceiver (Jaegle et al., 2021) architecture which also aims for a unified model across modalities and tasks.

In summary, Chameleon builds on a rich history of work in multimodal learning and token-based architectures, while pushing the boundaries in terms of model scale and architecture design. By demonstrating strong performance across a wide range of vision-language tasks and enabling new capabilities in mixed-modal reasoning and generation, Chameleon represents a significant step towards realizing the vision of general-

<span id="page-17-0"></span>
purpose multimodal foundation models.

## 7 Conclusion

In this paper, we introduced Chameleon, a new family of early-fusion token-based foundation models that set a new bar for multimodal machine learning. By learning a unified representation space over interleaved image and text tokens, Chameleon is a single model that achieves strong performance across a wide range of vision-language benchmarks while enabling new mixed-modal reasoning and generation capabilities.

The key to Chameleon’s success is its fully token-based architecture, which allows for seamless information integration across modalities. By quantizing images into discrete tokens and training on mixed-modal data from scratch, Chameleon learns to jointly reason over image and text in a way that is impossible with late-fusion architectures or models that maintain separate encoders for each modality. At the same time, Chameleon introduces novel techniques for stable and scalable training of early-fusion models, addressing key optimization and architectural design challenges that have previously limited the scale of such approaches. On tasks such as image captioning and visual question answering, Chameleon-34B outperforms models such as Flamingo and IDEFICS, while maintaining competitive performance on text-only benchmarks. Chameleon also unlocks entirely new possibilities for multimodal interaction, as demonstrated by its strong performance on our new benchmark for mixed-modal open-ended QA.

## Acknowledgements

We thank Naren Briar for her invaluable contribution to manually curating safety prompts, which were crucial for our safety tuning efforts. We also thank Pierre Fernandez for his indispensable support with the Chameleon release, Shelly Sheynin for her work on the Chameleon image tokenizer, Puxin Xu and David for helping us with datasets. Additionally, we thank Mitchell Wortsman for engaging in insightful discussions about stability in large-scale language models and Mike Lewis for general discussions and advice throughout the project. We thank Aaron Grattafiori, Firat Ozgenel, Divya Shah, Danny Livshits, Cristian Canton Ferrer, Saghar Hosseini, Ramon Calderer, Joshua Saxe, Daniel Song and Manish Bhatt for their help with the safety and red teaming efforts.

## Contributors

We attribute credit separated by bucket of work. Additionally, ∗ indicates joint first authors, † indicates key contributors, ‡ indicates workstream leads, and ♯ indicates project leads.

Pre-Training: Srinivasan Iyer∗, Bernie Huang∗, Lili Yu†, Arun Babu†, Chunting Zhou†, Kushal Tirumala, Xi Victoria Lin, Hu Xu, Xian Li, Akshat Shrivastava, Omer Levy‡, Armen Aghajanyan∗‡

Alignment and Safety: Ram Pasunuru∗, Andrew Cohen†, Aram H. Markosyan†, Koustuv Sinha†, Xiaoqing Ellen Tan†, Ivan Evtimov, Ping Yu, Tianlu Wang, Olga Golovneva, Asli Celikyilmaz‡

Inference and Evaluation: Pedro Rodriguez†, Leonid Shamis†, Vasu Sharma†, Christine Jou, Karthik Padthe†, Ching-Feng Yeh, Mingda Chen, Bapi Akula, Jacob Kahn‡, Daniel Li‡, Scott Yih‡

Overall Project: Barlas Oguz, Morteza Behrooz, Benjamin Muller, Carleigh Wood, Mary Williamson, Ramya Raghavendra, Barbara Usher, William Ngan, Nikolay Bashlykov, Lukas Blecher, Sony Theakanath (Lead PM), Ammar Rizvi (Lead TPM), Gargi Ghosh♯, Luke Zettlemoyer♯

<span id="page-18-0"></span>
## References

- Armen Aghajanyan, Bernie Huang, Candace Ross, Vladimir Karpukhin, Hu Xu, Naman Goyal, Dmytro Okhonko, Mandar Joshi, Gargi Ghosh, Mike Lewis, et al. Cm3: A causal masked multimodal model of the internet. arXiv preprint arXiv:2201.07520, 2022.
- Armen Aghajanyan, Lili Yu, Alexis Conneau, Wei-Ning Hsu, Karen Hambardzumyan, Susan Zhang, Stephen Roller, Naman Goyal, Omer Levy, and Luke Zettlemoyer. Scaling laws for generative mixed-modal language models. arXiv preprint arXiv:2301.03728, 2023.
- Agriflanders. Miniatuurpaardjes prijskamp - Agriflanders 2009, 2009. https://en.wikipedia.org/wiki/File: Miniatuurpaardje.jpg. CC-BY-SA 2.0, https://creativecommons.org/licenses/by/2.0/deed.en.
- Jean-Baptiste Alayrac, Jeff Donahue, Pauline Luc, Antoine Miech, Iain Barr, Yana Hasson, Karel Lenc, Arthur Mensch, Katherine Millican, Malcolm Reynolds, et al. Flamingo: a visual language model for few-shot learning. Advances in Neural Information Processing Systems, 35:23716–23736, 2022.
- Tadas Baltrušaitis, Chaitanya Ahuja, and Louis-Philippe Morency. Multimodal machine learning: A survey and taxonomy. IEEE transactions on pattern analysis and machine intelligence, 41(2):423–443, 2018.
- Hangbo Bao, Li Dong, Songhao Piao, and Furu Wei. Beit: Bert pre-training of image transformers. arXiv preprint arXiv:2106.08254, 2021.
- James Betker, Gabriel Goh, Li Jing, Tim Brooks, Jianfeng Wang, Linjie Li, Long Ouyang, Juntang Zhuang, Joyce Lee, Yufei Guo, et al. Improving image generation with better captions. Computer Science. https://cdn. openai. com/papers/dall-e-3. pdf, 2(3):8, 2023.
- Yonatan Bisk, Rowan Zellers, Jianfeng Gao, Yejin Choi, et al. Piqa: Reasoning about physical commonsense in natural language. In Proceedings of the AAAI conference on artificial intelligence, pages 7432–7439, 2020.
- Jun Chen, Han Guo, Kai Yi, Boyang Li, and Mohamed Elhoseiny. Visualgpt: Data-efficient adaptation of pretrained language models for image captioning. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 18030–18040, 2022.
- Xinlei Chen, Hao Fang, Tsung-Yi Lin, and Ramakrishna Vedantam. https://github.com/salaniz/pycocoevalcap, 2020.
- Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann, et al. PaLM: Scaling language modeling with pathways. arXiv preprint arXiv:2204.02311, 2022.
- Christopher Clark, Kenton Lee, Ming-Wei Chang, Tom Kwiatkowski, Michael Collins, and Kristina Toutanova. Boolq: Exploring the surprising difficulty of natural yes/no questions. arXiv preprint arXiv:1905.10044, 2019.
- Peter Clark, Isaac Cowhey, Oren Etzioni, Tushar Khot, Ashish Sabharwal, Carissa Schoenick, and Oyvind Tafjord. Think you have solved question answering? try arc, the ai2 reasoning challenge. arXiv preprint arXiv:1803.05457, 2018.
- Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano, et al. Training verifiers to solve math word problems. arXiv preprint arXiv:2110.14168, 2021.
- Mostafa Dehghani, Josip Djolonga, Basil Mustafa, Piotr Padlewski, Jonathan Heek, Justin Gilmer, Andreas Peter Steiner, Mathilde Caron, Robert Geirhos, Ibrahim Alabdulmohsin, et al. Scaling vision transformers to 22 billion parameters. In International Conference on Machine Learning, pages 7480–7512. PMLR, 2023.
- Oran Gafni, Adam Polyak, Oron Ashual, Shelly Sheynin, Devi Parikh, and Yaniv Taigman. Make-a-scene: Scene-based text-to-image generation with human priors. arXiv preprint arXiv:2203.13131, 2022.
- Gemini, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, et al. Gemini: a family of highly capable multimodal models. arXiv preprint arXiv:2312.11805, 2023.
- Yash Goyal, Tejas Khot, Douglas Summers-Stay, Dhruv Batra, and Devi Parikh. Making the V in VQA matter: Elevating the role of image understanding in visual question answering. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 6904–6913, 2017.

<span id="page-19-0"></span>
- Dan Hendrycks, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, and Jacob Steinhardt. Measuring massive multitask language understanding. In International Conference on Learning Representations, 2020.
- Dan Hendrycks, Collin Burns, Saurav Kadavath, Akul Arora, Steven Basart, Eric Tang, Dawn Song, and Jacob Steinhardt. Measuring mathematical problem solving with the math dataset. arXiv preprint arXiv:2103.03874, 2021.
- Or Honovich, Thomas Scialom, Omer Levy, and Timo Schick. Unnatural instructions: Tuning language models with (almost) no human labor, 2022.
- Drew A Hudson and Christopher D Manning. Gqa: A new dataset for real-world visual reasoning and compositional question answering. In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, pages 6700–6709, 2019.
- Andrew Jaegle, Sebastian Borgeaud, Jean-Baptiste Alayrac, Carl Doersch, Catalin Ionescu, David Ding, Skanda Koppula, Daniel Zoran, Andrew Brock, Evan Shelhamer, et al. Perceiver io: A general architecture for structured inputs & outputs. arXiv preprint arXiv:2107.14795, 2021.
- Albert Q Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, et al. Mistral 7b. arXiv preprint arXiv:2310.06825, 2023.
- Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, et al. Mixtral of experts. arXiv preprint arXiv:2401.04088, 2024.
- Yang Jin, Kun Xu, Liwei Chen, Chao Liao, Jianchao Tan, Bin Chen, Chenyi Lei, An Liu, Chengru Song, Xiaoqiang Lei, et al. Unified language-vision pretraining with dynamic discrete visual tokenization. arXiv preprint arXiv:2309.04669, 2023.
- Yuval Kirstain, Adam Polyak, Uriel Singer, Shahbuland Matiana, Joe Penna, and Omer Levy. Pick-a-pic: An open dataset of user preferences for text-to-image generation, 2023.
- Klaus Krippendorff. Content analysis: An introduction to its methodology. Sage publications, 2018.
- Taku Kudo and John Richardson. Sentencepiece: A simple and language independent subword tokenizer and detokenizer for neural text processing. arXiv preprint arXiv:1808.06226, 2018.
- Hugo Laurençon, Lucile Saulnier, Léo Tronchon, Stas Bekman, Amanpreet Singh, Anton Lozhkov, Thomas Wang, Siddharth Karamcheti, Alexander M Rush, Douwe Kiela, et al. Obelisc: An open web-scale filtered dataset of interleaved image-text documents. arXiv preprint arXiv:2306.16527, 2023.
- Kevin Lee and Shubho Sengupta. Introducing the ai research supercluster — meta’s cutting-edge ai supercomputer for ai research. https://ai.facebook.com/blog/ai-rsc/, 2022.
- Benjamin Lefaudeux, Francisco Massa, Diana Liskovich, Wenhan Xiong, Vittorio Caggiano, Sean Naren, Min Xu, Jieru Hu, Marta Tintore, Susan Zhang, Patrick Labatut, Daniel Haziza, Luca Wehrstedt, Jeremy Reizenstein, and Grigory Sizov. xformers: A modular and hackable transformer modelling library. https://github.com/facebookresearch/ xformers, 2022.
- Tsung-Yi Lin, Michael Maire, Serge Belongie, James Hays, Pietro Perona, Deva Ramanan, Piotr Dollár, and C Lawrence Zitnick. Microsoft coco: Common objects in context. In European conference on computer vision, pages 740–755. Springer, 2014.
- Haotian Liu, Chunyuan Li, Yuheng Li, and Yong Jae Lee. Improved baselines with visual instruction tuning. arXiv preprint arXiv:2310.03744, 2023a.
- Haotian Liu, Chunyuan Li, Qingyang Wu, and Yong Jae Lee. Visual instruction tuning. arXiv preprint arXiv:2304.08485, 2023b.
- Ze Liu, Yutong Lin, Yue Cao, Han Hu, Yixuan Wei, Zheng Zhang, Stephen Lin, and Baining Guo. Swin transformer: Hierarchical vision transformer using shifted windows. In Proceedings of the IEEE/CVF international conference on computer vision, pages 10012–10022, 2021.
- Ilya Loshchilov and Frank Hutter. Decoupled weight decay regularization. arXiv preprint arXiv:1711.05101, 2017.
- Giacomo Marzi, Marco Balzano, and Davide Marchiori. K-alpha calculator–krippendorff’s alpha calculator: A userfriendly tool for computing krippendorff’s alpha inter-rater reliability coefficient. MethodsX, 12:102545, 2024. ISSN

<span id="page-20-0"></span>
- 2215-0161. doi: https://doi.org/10.1016/j.mex.2023.102545. https://www.sciencedirect.com/science/article/pii/ S2215016123005411.
- Todor Mihaylov, Peter Clark, Tushar Khot, and Ashish Sabharwal. Can a suit of armor conduct electricity? a new dataset for open book question answering. arXiv preprint arXiv:1809.02789, 2018.
- OpenAI. GPTV System Card. https://cdn.openai.com/papers/GPTV\_System\_Card.pdf, 2023.
- Adam Paszke, Sam Gross, Francisco Massa, Adam Lerer, James Bradbury, Gregory Chanan, Trevor Killeen, Zeming Lin, Natalia Gimelshein, Luca Antiga, et al. PyTorch: An imperative style, high-performance deep learning library. In NeurIPS, 2019.
- Bryan A Plummer, Liwei Wang, Chris M Cervantes, Juan C Caicedo, Julia Hockenmaier, and Svetlana Lazebnik. Flickr30k entities: Collecting region-to-phrase correspondences for richer image-to-sentence models. In Proceedings of the IEEE international conference on computer vision, pages 2641–2649, 2015.
- Aditya Ramesh, Mikhail Pavlov, Gabriel Goh, Scott Gray, Chelsea Voss, Alec Radford, Mark Chen, and Ilya Sutskever. Zero-shot text-to-image generation. arXiv preprint arXiv:2102.12092, 2021.
- Aditya Ramesh, Prafulla Dhariwal, Alex Nichol, Casey Chu, and Mark Chen. Hierarchical text-conditional image generation with clip latents. arXiv preprint arXiv:2204.06125, 2022.
- Baptiste Roziere, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Tal Remez, Jérémy Rapin, et al. Code llama: Open foundation models for code. arXiv preprint arXiv:2308.12950, 2023.
- Keisuke Sakaguchi, Ronan Le Bras, Chandra Bhagavatula, and Yejin Choi. Winogrande: An adversarial winograd schema challenge at scale. Communications of the ACM, 64(9):99–106, 2021.
- Mikayel Samvelyan, Sharath Chandra Raparthy, Andrei Lupu, Eric Hambro, Aram H. Markosyan, Manish Bhatt, Yuning Mao, Minqi Jiang, Jack Parker-Holder, Jakob Foerster, Tim Rocktäschel, and Roberta Raileanu. Rainbow teaming: Open-ended generation of diverse adversarial prompts, 2024.
- Maarten Sap, Hannah Rashkin, Derek Chen, Ronan LeBras, and Yejin Choi. Socialiqa: Commonsense reasoning about social interactions. arXiv preprint arXiv:1904.09728, 2019.
- Rylan Schaeffer. Pretraining on the test set is all you need. arXiv preprint arXiv:2309.08632, 2023.
- Christoph Schuhmann, Romain Beaumont, Richard Vencu, Cade Gordon, Ross Wightman, Mehdi Cherti, Theo Coombes, Aarush Katta, Clayton Mullis, Mitchell Wortsman, et al. Laion-5b: An open large-scale dataset for training next generation image-text models. arXiv preprint arXiv:2210.08402, 2022.
- Georges Seguin. Mille-feuille, 2010. https://en.wikipedia.org/wiki/File:Mille-feuille\_20100916.jpg. CC-BY-SA 3.0, https://creativecommons.org/licenses/by-sa/3.0/deed.en.
- Rico Sennrich, Barry Haddow, and Alexandra Birch. Neural machine translation of rare words with subword units. In ACL, Berlin, Germany, 2016. https://aclanthology.org/P16-1162.
- ShareGPT. GPTV System Card. https://sharegpt.com/, 2023.
- Noam Shazeer. Glu variants improve transformer. arXiv preprint arXiv:2002.05202, 2020.
- Maksim Sokolov. Sagrada Familia July 2022, 2022. https://en.wikipedia.org/wiki/File:Sagrada\_Familia\_%28July\_ 2022%29\_08.jpg. CC-BY-SA-4.0, https://creativecommons.org/licenses/by-sa/4.0/deed.en.
- Nitish Srivastava, Geoffrey Hinton, Alex Krizhevsky, Ilya Sutskever, and Ruslan Salakhutdinov. Dropout: a simple way to prevent neural networks from overfitting. The journal of machine learning research, 15(1):1929–1958, 2014.
- Jianlin Su, Yu Lu, Shengfeng Pan, Ahmed Murtadha, Bo Wen, and Yunfeng Liu. Roformer: Enhanced transformer with rotary position embedding. arxiv e-prints, art. arXiv preprint arXiv:2104.09864, 2021.
- Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, et al. Llama 2: Open foundation and fine-tuned chat models. arXiv preprint arXiv:2307.09288, 2023.
- Jarek Tuszyński. American toad (Bufo americanus) found in Fairfax, Virginia, 2015. https://en.wikipedia.org/wiki/File: Miniatuurpaardje.jpg. CC-BY-SA-4.0, https://creativecommons.org/licenses/by-sa/4.0/deed.en.
- Ramakrishna Vedantam, C Lawrence Zitnick, and Devi Parikh. Cider: Consensus-based image description evaluation. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 4566–4575, 2015.

<span id="page-21-0"></span>
- Mitchell Wortsman, Peter J Liu, Lechao Xiao, Katie Everett, Alex Alemi, Ben Adlam, John D Co-Reyes, Izzeddin Gur, Abhishek Kumar, Roman Novak, et al. Small-scale proxies for large-scale transformer training instabilities. arXiv preprint arXiv:2309.14322, 2023.
- Lili Yu, Bowen Shi, Ramakanth Pasunuru, Benjamin Muller, Olga Golovneva, Tianlu Wang, Arun Babu, Binh Tang, Brian Karrer, Shelly Sheynin, et al. Scaling autoregressive multi-modal models: Pretraining and instruction tuning. arXiv preprint arXiv:2309.02591, 2023.
- Rowan Zellers, Ari Holtzman, Yonatan Bisk, Ali Farhadi, and Yejin Choi. Hellaswag: Can a machine really finish your sentence? arXiv preprint arXiv:1905.07830, 2019.
- Biao Zhang and Rico Sennrich. Root mean square layer normalization. Advances in Neural Information Processing Systems, 32, 2019.
- Chunting Zhou, Pengfei Liu, Puxin Xu, Srini Iyer, Jiao Sun, Yuning Mao, Xuezhe Ma, Avia Efrat, Ping Yu, Lili Yu, et al. Lima: Less is more for alignment. arXiv preprint arXiv:2305.11206, 2023.

<span id="page-22-0"></span>
## Appendix

## A Samples

![](assets/_page_22_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_22_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:19:40
> 
> ---
> 
> The image is a photograph of a black and white dog with a red collar. The dog is looking slightly to the left of the camera. The background appears to be an outdoor setting with green grass. The image also contains a small text annotation in the bottom left corner that reads "Generated by AI Research at Meta."


Prompt: My mom just adopted a dog called a mountain cur. I thought I knew a lot about dogs but I have never heard of this breed. Can you give me some basic information as well as show me a few pictures of what they look like?

Response: Sure! Mountain Cur dogs are a type of hunting dog that originated in the Appalachian Mountains. They are known for their intelligence, loyalty, and strong prey drive. They are typically medium-sized dogs, with a muscular build and a short, easy-to-maintain coat that comes in a variety of colors including black, brown, tan, red, and white. Here is an image of a black and white mountain cur dog: <img>

They have a long muzzle, floppy ears, and a bushy tail. They are generally friendly and outgoing, but can be wary of strangers if not socialized properly. They are highly energetic dogs that require regular exercise and mental stimulation to prevent boredom and destructive behavior.

![](assets/_page_22_Figure_1.jpg)

> **AI Description:**
> **Source:** `assets/_page_22_Figure_1.jpg`
> 
> **Generated:** 2026-05-16 20:19:45
> 
> ---
> 
> The image is a photograph of a dog standing on grass in a forested area. The dog has a black and brown coat with a distinctive black patch on its back. The background shows trees and greenery, suggesting an outdoor setting. The bottom left corner of the image contains text that reads "Generated by AI Research at Meta."


Here is another image of a mountain cur dog in the forest: <img>

Mountain Cur dogs are often used for hunting, particularly for small game such as squirrels, raccoons, and rabbits. They are also popular family pets due to their loyal and affectionate nature. However, they do require regular exercise and training to ensure they remain happy and healthy. With proper care and attention, Mountain Cur dogs can make wonderful companions for active families or individuals who enjoy the outdoors.

![](assets/_page_22_Figure_2.jpg)

> **AI Description:**
> **Source:** `assets/_page_22_Figure_2.jpg`
> 
> **Generated:** 2026-05-16 20:19:34
> 
> ---
> 
> The image is a photograph of a polar bear standing on ice. The bear is white with a thick coat of fur and appears to be in a natural environment, likely the Arctic. The ice beneath the bear is light blue, indicating it is frozen water. The bear's posture suggests it is walking or moving forward. There is a small text in the bottom left corner that reads "Generated by AI models at Meta," which is likely a watermark or credit for the image generation.


Prompt: What color is a polar bear’s fur? Show me a photograph of the polar bear in the wild.

Response: Polar bears have white fur, which helps them blend in with their snowy and icy surroundings. This camouflage allows them to hunt more effectively, as they can sneak up on their prey without being detected. Additionally, the white fur reflects sunlight, keeping the polar bear cool in the harsh Arctic environment. Here is an image of a polar bear in the wild:<img>

<span id="page-23-0"></span>
![](assets/_page_23_Figure_0.jpg)

> **AI Description:**
> **Source:** `assets/_page_23_Figure_0.jpg`
> 
> **Generated:** 2026-05-16 20:19:29
> 
> ---
> 
> The image is a photograph of a potted plant, specifically a pothos, with variegated leaves that have a mix of green and yellowish-white patterns. The plant is placed in a ceramic pot, which is sitting on a decorative plate. The background is dark, which makes the plant stand out prominently. The image captures the natural beauty and texture of the leaves, highlighting the unique patterns and colors.


Prompt: <img> My plant looks like this... Is this a problem? If so, how do I fix it?

Response: The image you provided shows a pothos plant with yellowing leaves, which can be caused by a variety of factors, including overwatering, underwatering, lack of nutrients, too much direct sunlight, or pests. Here are some steps you can take to troubleshoot and potentially fix the issue:

• Check the soil: Pothos plants prefer well-draining soil that is neither too wet nor too dry. If the soil feels soggy or waterlogged, stop watering for a while. Conversely, if the soil feels dry to the touch, it’s time to water.

• Adjust watering: Overwatering is a common cause of yellowing leaves on pothos plants. Reduce the frequency of watering and allow the soil to dry out slightly between waterings.

• Provide adequate light: Pothos plants prefer bright, indirect light but can tolerate low light conditions. If your plant is not getting enough light, move it to a brighter location.

• Fertilize: Pothos plants benefit from regular fertilization. Use a balanced, water-soluble fertilizer and follow the instructions on the label.

• Prune: Yellow leaves can be removed to improve the appearance of the plant. Cut back any yellow leaves to the nearest healthy stem using clean pruning shears.

• Check for pests: Inspect the plant for pests such as mealybugs, spider mites, or scale. If present, treat the plant with insecticidal soap or neem oil according to the product’s instructions.

• Repot: If the plant is pot-bound (meaning the roots have filled the container), consider repotting it in a larger container with fresh, well-draining soil.

By following these steps, you may be able to identify and address the underlying cause of the yellowing leaves on your pothos plant.

## B Additional Information of Human Evaluations

Table 8 Descriptions of the prompt task categories.
![](tables/table_pg23_num0.csv)

For the twelve task categories of the prompts we collected for human evaluation, a short description of each category can be found in Table 8.

The task fulfillment rates, broken down by each task category and modality are shown in Table 9 and Table 10. Chameleon’s win rates, broken down by task category and modality, are shown in Table 11, Table 12, Table 13 and Table 14.

<span id="page-24-0"></span>
Table 9 Task fulfillment breakdown.
![](tables/table_pg24_num0.csv)

![](tables/table_pg24_num1.csv)

Table 10 Modality fulfillment breakdown.
![](tables/table_pg24_num2.csv)

<span id="page-25-0"></span>
Table 11 Complete Win Rates: Chameleon vs. Gemini+.
![](tables/table_pg25_num0.csv)

Table 12 Complete Win Rates: Chameleon vs. GPT-4V+.
![](tables/table_pg25_num1.csv)

<span id="page-26-0"></span>
Table 13 Complete Win Rates: Chameleon vs. Gemini.
![](tables/table_pg26_num0.csv)

Table 14 Complete Win Rates: Chameleon vs. $\mathrm { G P T - 4 V } .$
![](tables/table_pg26_num1.csv)
