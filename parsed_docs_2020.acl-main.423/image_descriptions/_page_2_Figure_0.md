# _page_2_Figure_0.jpg

**Source:** `assets/_page_2_Figure_0.jpg`

**Generated:** 2026-05-15 14:11:55

---

The image contains a comparison between two models: BERT and SenseBERT. It is a diagram illustrating the architecture of these models, specifically focusing on the input and output components. The diagram includes:

1. **Input Representation**: Both models take in a sequence of tokens represented as \( x^{(1)} \) to \( x^{(N)} \), with a special token "[MASK]" included. SenseBERT also includes a separate input \( S \) for sense information.

2. **Transformation**: The input tokens are transformed using a weight matrix \( W \) and a sense matrix \( S \). The transformed representations are denoted as \( Wx^{(j)} \) and \( SMx^{(j)} \) for BERT and SenseBERT, respectively.

3. **Model Output**: Both models pass the transformed inputs through a Transformer encoder. The output from BERT is denoted as \( y^{words} \), while SenseBERT produces two outputs: \( y^{words} \) and \( y^{senses} \), where \( y^{senses} \) is the output for sense information.

The diagram highlights the differences in the input representations and the resulting outputs, emphasizing the integration of sense information in SenseBERT.