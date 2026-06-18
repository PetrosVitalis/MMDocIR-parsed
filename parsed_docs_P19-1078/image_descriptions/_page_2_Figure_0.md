# _page_2_Figure_0.jpg

**Source:** `assets/_page_2_Figure_0.jpg`

**Generated:** 2026-05-30 10:34:43

---

The image is a technical diagram illustrating a dialogue system architecture, specifically for a conversational agent. It includes:

1. **Utterance Encoder**: This part of the diagram shows how utterances are encoded into a context vector \( c_{j0} \). The encoder processes a sequence of utterances and generates a representation that captures the user's input.

2. **Slot Gate \( G_j \)**: This component decides which slots to focus on based on the context vector. It has three options: "PTR" (Pointer), "DONTCARE", and "NONE". The "PTR" option is highlighted, indicating it is the active slot gate.

3. **State Generator**: This section generates the final state of the dialogue, which includes the user's name, domain (e.g., hotel, train, attraction), and slot values (e.g., price, area, day, departure, name, leave at, food, etc.). The state generator uses the context vector and the utterance encoder's output to predict the final dialogue state.

4. **Transition Probabilities**: The diagram also includes transition probabilities \( P_{j0}^{\text{final}} \) and \( P_{j0}^{\text{gen}} \), which are used to compute the final state probabilities and the generated state probabilities, respectively.

The diagram is labeled with example utterances from a dialogue between a bot and a user, illustrating how the system processes and responds to user inputs.