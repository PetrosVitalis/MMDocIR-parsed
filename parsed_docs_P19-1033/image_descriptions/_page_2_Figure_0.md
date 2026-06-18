# _page_2_Figure_0.jpg

**Source:** `assets/_page_2_Figure_0.jpg`

**Generated:** 2026-05-30 10:17:35

---

The image is a diagram illustrating a neural network architecture for news topic and subtopic prediction. It includes several key components:

1. **Input Layers**: The diagram shows the input layers, which include "Subtopic Embedding," "Topic Embedding," and "Word Embedding." These are represented by boxes with arrows pointing to them, indicating the flow of data.

2. **Word Embedding Layer**: This layer is shown at the bottom of the diagram, with a series of boxes labeled \( w_1, w_2, \ldots, w_N \), representing word embeddings for a news title.

3. **Subtopic and Topic Embeddings**: These are fed into the network, with the subtopic embedding \( e_{sv} \) and topic embedding \( e_v \) being combined and then passed to the next layer.

4. **Attention Mechanism**: The diagram includes an attention mechanism, represented by the orange box labeled \( e_t \) and the dashed lines connecting it to the word embeddings \( c_1, c_2, \ldots, c_N \). This mechanism is used to focus on specific parts of the input sequence.

5. **Output Layer**: The final output \( a_1, a_2, \ldots, a_N \) represents the predicted subtopic and topic scores for each word in the news title.

The diagram effectively visualizes the flow of data through the neural network, highlighting the use of word embeddings, subtopic and topic embeddings, and an attention mechanism to predict news topics and subtopics.