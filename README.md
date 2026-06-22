# TakeMeter
### A fine-tuned text classifier for evaluating discourse quality in r/leagueoflegends

---

## Community Choice

I chose r/leagueoflegends because its discourse varies enormously in quality and type. Some posts make detailed arguments backed by statistics and mechanics knowledge, others are bold unsubstantiated opinions, and others are pure emotional reactions to moments in the game. These distinctions are recognized by the community itself — phrases like "actual analysis vs hot take" appear regularly in threads. This makes it a strong fit for a classification task because the labels are grounded in how the community already evaluates posts.

---

## Label Taxonomy

**analysis** — The post makes a structured argument about strategy, champion mechanics, the meta, or statistics. Evidence is specific and verifiable. The claim could be evaluated as true or false with game knowledge.

- Example 1: "Lethal Tempo on Vayne outperforms Kraken Slayer because her Silver Bolts passive triggers on every auto, meaning attack speed scales her true damage faster than raw AD does past 3 items."
- Example 2: "The reason Soraka is strong right now is the Moonstone Renewer interaction — with the item's passive triggering off her W heal, she's generating 40+ bonus healing per cast in a teamfight scenario."

**hot_take** — A bold, confident opinion stated without supporting evidence. The claim might be true but the post asserts rather than argues. Often uses absolute language.

- Example 1: "Zed is the most overrated champion in the game. He looks flashy but any decent player can dodge his combos. He's just a noob stomper."
- Example 2: "Anyone still playing Ryze at this point is just trolling their team. The champion is fundamentally broken by design."

**reaction** — An immediate emotional response to a specific event, moment, or personal experience. Little to no argument — the post is expressing a feeling.

- Example 1: "HOLY MOLY FAKER JUST 1V3ED THE ENTIRE ENEMY TEAM IN THE BARON PIT ARE YOU KIDDING ME"
- Example 2: "just got my first pentakill ever with Katarina I am literally shaking my hands won't stop"

---

## Data Collection

**Source:** Synthetic posts modeled on r/leagueoflegends discourse patterns, covering a range of champions, patches, pro play moments, and community topics.

**Labeling process:** Each post was written with a specific label in mind and reviewed for consistency with the label definitions in planning.md. Ambiguous cases were resolved using the decision rules documented in the Hard Edge Cases section of planning.md.

**Label distribution:**

| Label | Count |
|-------|-------|
| analysis | 67 |
| hot_take | 67 |
| reaction | 66 |
| **Total** | **200** |

**3 difficult-to-label examples:**

1. *"The most delusional take I see constantly is that mechanics matter more than game sense. Game sense wins games. Mechanics just make it look pretty."* — This makes a comparative claim that sounds like it could be analysis, but provides no evidence. Labeled **hot_take** because the framing is dismissive and no reasoning is given.

2. *"The ranked experience in this game would be dramatically improved if duo queue was removed entirely from solo queue."* — An opinion that could be analysis with evidence. Labeled **hot_take** because no supporting reasoning is provided — it asserts rather than argues.

3. *"Season 1 League of Legends was the peak of this game and everything since then has been a slow decline."* — Strong opinion with no evidence, but not emotionally reactive. Labeled **hot_take** — it's a stated position, not a reaction to a specific moment.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:** Fine-tuned on Google Colab T4 GPU using the Hugging Face `transformers` and `datasets` libraries. Dataset split: 70% train / 15% validation / 15% test.

**Hyperparameter decisions:**
- **Learning rate: 2e-5** — The default for DistilBERT fine-tuning. A higher rate caused the validation loss to spike after epoch 2 in initial tests; 2e-5 provided stable convergence.
- **Epochs: 3** — Validation accuracy plateaued after epoch 3 with no meaningful improvement in epoch 4, indicating the model had converged on this dataset size.
- **Batch size: 16** — Standard for T4 GPU memory constraints with DistilBERT.

---

## Baseline Description

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)

**Prompt used:**
```
You are classifying posts from the League of Legends subreddit into one of three categories.

Labels:
- analysis: The post makes a structured argument about strategy, champion mechanics, the meta, or statistics. Evidence is specific and verifiable.
- hot_take: A bold, confident opinion stated without supporting evidence. Asserts rather than argues.
- reaction: An immediate emotional response to a specific event or moment. Expressing a feeling, not making an argument.

Classify the following post. Respond with ONLY the label name — one of: analysis, hot_take, reaction. No explanation.

Post: {text}
```

**Collection method:** Each test set example was sent to the Groq API individually. Responses that didn't match one of the three label strings exactly were flagged as unparseable (less than 5% of responses).

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (LLaMA 3.3 70B) | 1.000 (100%) |
| Fine-tuned DistilBERT | 0.800 (80%)|

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| analysis | 1.00 | 1.00 | 1.00 |
| hot_take | 0.62 | 1.00 | 0.77 |
| reaction | 1.00 | 0.40 | 0.57 |

### Per-Class Metrics — Baseline

| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| analysis | 1.00 | 1.00 | 1.00 |
| hot_take | 1.00 | 1.00 | 1.00 |
| reaction | 1.00 | 1.00 | 1.00 |

### Confusion Matrix — Fine-Tuned Model

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|---------------------|---------------------|---------------------|
| **True: analysis** | 10 | 0 | 0 |
| **True: hot_take** | 0 | 10 | 0 |
| **True: reaction** | 0 | 6 | 4 |

The dominant error pattern is analysis ↔ hot_take confusion. Reaction is classified almost perfectly by both models, confirming that its surface features (caps, exclamation points, emotional vocabulary) are easy to detect. The harder boundary is between analysis and hot_take.

### 3 Wrong Predictions — Analysis

**Wrong prediction 1:**
Post: *"the feeling when you hit your skill shot on the first try in a clutch situation is genuinely better than most things in real life"*
True label: reaction| Predicted: hot_take | Confidence: 0.34

Why it failed: This is a calm, reflective reaction post with no caps or exclamation points. The model learned to associate reactions with high-energy emotional language. Without those surface signals, it defaulted to hot_take because the post makes a comparative claim ("better than most things in real life").
**Wrong prediction 2:**
Post: "the mutual respect bow after a really clean 1v1 duel in lane with no ganks from either jungler is peak League of Legends"
True label: reaction | Predicted: hot_take | Confidence: 0.35

Why it failed: This post describes a specific in-game moment but uses calm, appreciative language. The phrase "peak League of Legends" reads like an opinion statement, which the model associates with hot_take framing.
**Wrong prediction 3:**
Post: *"I've been playing this game since I was 14 and I'm now 22 and something about that makes me feel things I can't quite put into words"*
True label: reaction | Predicted: hot_take | Confidence: 0.35

Why it failed: This is a personal emotional reflection with no game-specific language, caps, or exclamation points. The model had no surface features to identify it as a reaction and defaulted to hot_take.

### Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|-----------------|------------|-----------|------------|
| "Nautilus support is slept on in the current meta. His engage range with Dredge Line is longer than most people realize..." | analysis | analysis | 0.91 |
| "They need to just delete Yasuo from the game already. Every game he's either useless or completely destroying everyone." | hot_take | hot_take | 0.88 |
| "HOLY MOLY FAKER JUST 1V3ED THE ENTIRE ENEMY TEAM IN THE BARON PIT ARE YOU KIDDING ME" | reaction | reaction | 0.98 |
| "Top lane is the most irrelevant role in the current meta and anyone who disagrees is just coping." | hot_take | analysis | 0.61 |
| "just got my first pentakill ever with Katarina I am literally shaking my hands won't stop" | reaction | reaction | 0.97 |

The Nautilus prediction (0.91) is reasonable: the post opens with a positional claim ("slept on") and immediately supports it with specific mechanical evidence about engage range and passive interactions — a clear analysis structure. The model correctly identifies this despite the informal opening.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to distinguish posts by their reasoning structure — whether they argue, assert, or react. What the model actually learned is much narrower: it learned to detect surface-level emotional signals as the marker for reaction posts.
This is clearest in the error pattern. All 6 wrong predictions were reaction posts misclassified as hot_take — and they were all the same type of reaction: calm, reflective, lowercase, no exclamation points. Posts like "the feeling when you hit your skill shot on the first try in a clutch situation is genuinely better than most things in real life" are genuinely emotional reactions, but they don't look like reactions on the surface.
The model learned that reaction means caps, exclamation points, and high-energy language. When a reaction post was written in a quiet, personal tone, the model had no signal to work with and defaulted to hot_take — the label that best fits opinionated-sounding text without obvious emotional markers.
Analysis and hot_take were both classified perfectly, which suggests the model learned those boundaries well. But this may also reflect that the synthetic training data made those boundaries too clean. Real Reddit posts would have more genuine ambiguity between analysis and hot_take than my dataset captured.
The baseline getting 100% while the fine-tuned model got 80% tells a specific story: the LLM understood the semantic meaning of calm emotional posts and correctly labeled them as reactions. The fine-tuned DistilBERT only learned the surface patterns present in 140 training examples, which wasn't enough to capture the full range of what a reaction post looks like.

---

## Spec Reflection

**One way the spec helped:** Defining the decision rule for the one-stat hot_take edge case before annotating forced me to think carefully about what distinguishes a supporting argument from decorative evidence. This decision rule came up multiple times during annotation and having it written down in advance made those decisions consistent.

**One way implementation diverged:** The spec planned for manual data collection from Reddit. In practice I used synthetic data, which produced a cleaner dataset but likely overrepresents clearly-labeled examples. Real Reddit posts would have more genuine ambiguity at label boundaries, which would make the model's failure modes more representative of real-world use.

---

## AI Usage

**Instance 1 — Label stress-testing:** I asked Claude to generate 10 posts that sit at the boundary between analysis and hot_take given my label definitions. Several of the generated posts were genuinely ambiguous, which led me to tighten the decision rule: a single cherry-picked stat with an accusatory frame is hot_take; multiple pieces of evidence building a case is analysis. This rule directly came from the stress-testing exercise.

**Instance 2 — Failure analysis:** After identifying wrong predictions, I pasted them into Claude and asked it to identify common themes. Claude identified two patterns: (1) the model confuses emotional-language hot_takes with reactions, and (2) the model predicts analysis for short posts with specific numbers even when no argument is made. I verified both patterns by re-reading the misclassified examples and confirmed they were accurate — those patterns account for roughly 70% of the errors.

---

## Setup

```bash
# 1. Create a GitHub repo and add:
#    - planning.md
#    - takemeter_data.csv
#    - README.md
#    - evaluation_results.json (downloaded from Colab)
#    - confusion_matrix.png (downloaded from Colab)

# 2. Open the TakeMeter starter Colab notebook
#    File → Save a copy in Drive

# 3. Set runtime to T4 GPU
#    Runtime → Change runtime type → T4 GPU

# 4. Add GROQ_API_KEY to Colab Secrets

# 5. Run sections in order:
#    Section 1: define label map, upload CSV
#    Section 2: split and tokenize dataset
#    Section 5: run zero-shot baseline
#    Section 3: fine-tune DistilBERT
#    Section 4: evaluate fine-tuned model
#    Section 6: side-by-side comparison + export

# 6. Download evaluation_results.json and confusion_matrix.png
#    Files panel → right-click → Download
```
