# r/beginnerfitness Text Classifier

A fine-tuned DistilBERT model that classifies posts from r/beginnerfitness into three categories: **Scientific**, **Experiential**, and **Motivational**. Built as a demonstration of how fine-tuning a small transformer on community-specific data dramatically outperforms zero-shot prompting for a nuanced, domain-specific classification task.

---

## What I Built

A text classifier trained on 348 manually labeled posts collected from r/beginnerfitness via the Apify Reddit scraper. The model distinguishes between posts that make evidence-based claims (Scientific), posts rooted in personal experience and gym lore (Experiential), and posts focused on mindset, etiquette, and milestones (Motivational).

The baseline is a zero-shot Groq LLM classifier using a structured system prompt with label definitions and one example per label. The fine-tuned model is DistilBERT (`distilbert-base-uncased`) trained for 3 epochs on the labeled dataset.

---

## Label Definitions

**Scientific:** Posts that rely on established training methodologies, biomechanics, nutritional science (e.g., macro splits, caloric deficits), or specific periodization math (e.g., RPE, percentages of 1-rep max) to answer a question or make a claim.

**Experiential:** Posts that offer advice or draw conclusions based strictly on personal experience (n=1), gym lore, or the sheer volume of time spent training, often disregarding optimal science in favor of what practically worked for the poster.

**Motivational:** Posts focused on the psychological aspects of fitness — discipline, gym etiquette, overcoming mental blocks, rants about other gym-goers, or celebrating personal milestones — rather than the mechanics or science of training.

---

## Evaluation Results

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq) | 34.0% |
| Fine-tuned DistilBERT | **71.7%** |
| Improvement | +37.7 pp |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Scientific | 0.71 | 0.63 | 0.67 | 19 |
| Experiential | 0.83 | 0.90 | **0.86** | 21 |
| Motivational | 0.54 | 0.54 | 0.54 | 13 |
| **Macro avg** | 0.69 | 0.69 | **0.69** | 53 |
| Weighted avg | 0.71 | 0.72 | 0.71 | 53 |

### Per-Class Metrics — Zero-Shot Baseline (Groq)

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Scientific | 0.78 | 0.37 | 0.50 | 19 |
| Experiential | 0.29 | 0.33 | 0.31 | 21 |
| Motivational | 0.20 | 0.31 | 0.24 | 13 |
| **Macro avg** | 0.42 | 0.34 | **0.35** | 53 |
| Weighted avg | 0.44 | 0.34 | 0.36 | 53 |

### Confusion Matrix — Fine-Tuned Model

Rows = true label, columns = predicted label.

| | Predicted: Scientific | Predicted: Experiential | Predicted: Motivational |
|---|---|---|---|
| **True: Scientific** | 12 | 2 | 5 |
| **True: Experiential** | 1 | 19 | 1 |
| **True: Motivational** | 4 | 2 | 7 |

The dominant failure pattern is the **Scientific ↔ Motivational boundary**: 5 Scientific posts were predicted Motivational and 4 Motivational posts were predicted Scientific. Experiential is the model's strongest class (19/21 correct, F1 = 0.86), reflecting that personal-experience language is the most linguistically distinct of the three.

---

## Failure Analysis

**15 of 53 test examples were misclassified.** Before writing this analysis, the full list of wrong predictions was reviewed with Claude to surface patterns. Claude identified three recurring themes: (1) Motivational posts that mention a specific exercise or supplement get pulled toward Scientific, (2) Scientific posts written in a personal, narrative voice get pulled toward Motivational or Experiential, and (3) short or low-information posts are unreliable regardless of true label. All three patterns held up on manual review of the examples.

### Failure 1 — Motivational → Scientific (annotation inconsistency)

> *"Same, I noticed I wasn't having the mid afternoon crash and more mental clarity when I started taking creatine."*
> **True:** Motivational | **Predicted:** Scientific | **Confidence:** 0.98

The model predicted Scientific with near-certainty because the post mentions creatine, a supplement strongly associated with scientific discourse in the training data. However, the post contains no scientific claim — it is a personal observation shared as social validation, which fits Motivational under the definition (celebrating a personal outcome). This is a **labeling problem**: the annotator correctly identified the intent but the topic-level signal (creatine) overwhelmed the structural signal (personal milestone). The model learned the wrong shortcut. A fix would be to include more Motivational training examples that mention supplements or specific lifts to show the model that topic alone does not determine the label.

### Failure 2 — Scientific → Motivational (annotation inconsistency)

> *"I'm 56, just started going to the gym very regularly a year ago. I'm very self-conscious and not a jock, so it took a lot to get there — but I love it, and a nice morning workout sets up my whole day..."*
> **True:** Scientific | **Predicted:** Motivational | **Confidence:** 0.97

This is a strong case where the **model is arguably more correct than the label**. The post is a personal milestone story with no reference to training methodology, periodization, or nutritional science — it matches the Motivational definition almost exactly. The annotation appears to be an error: the post was likely labeled Scientific because it appeared in a thread about training frequency, but the post itself contains no scientific content. This inflates the Scientific false-negative count artificially. It illustrates why annotation should be judged on the post text alone, not the thread context.

### Failure 3 — Experiential → Scientific (boundary ambiguity)

> *"I don't use this. I just make my daily LISS cardio 30 minutes after wake-up with 7 km/h. But for your question — it depends on bodyweight. For me it's ~95–125 bpm. But if you just do low intensity ca..."*
> **True:** Experiential | **Predicted:** Scientific | **Confidence:** 0.98

The post uses specific numbers (heart rate range, speed, duration) which are surface features the model associates with Scientific content. But the numbers are entirely self-referential ("for me it's ~95–125 bpm") — they describe the poster's own practice, not a claim derived from exercise science. This is a **boundary problem**, not a labeling error: the post is correctly labeled Experiential, but its numerical precision mimics the language of Scientific posts. The hard edge case rule in planning.md (personal outcome = Experiential regardless of vocabulary) was applied correctly here, but the model has no access to that rule.

---

## Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "If your goal is hypertrophy, literature suggests you need to hit each muscle group with 10–20 hard sets per week, keeping your RIR between 1 and 3." | Scientific | **Scientific** ✓ | 0.96 |
| "Honestly, don't overthink the spreadsheets. I ate a dozen eggs a day, drank a gallon of milk, ran linear progression until I puked, and put 50lbs on my squat in a month." | Experiential | **Experiential** ✓ | 0.94 |
| "Four months is not a reasonable amount of time to expect to see significant visible changes... that's a good rep range for a beginner..." | Experiential | Motivational ✗ | 0.96 |
| "Same, I noticed I wasn't having the mid afternoon crash and more mental clarity when I started taking creatine." | Motivational | Scientific ✗ | 0.98 |
| "I'm a beginner and have been looking at these posts... I'm overwhelmed and just want to get a PT. How do I choose one that's right for me?" | Motivational | Experiential ✗ | 0.94 |

The first example (Scientific, confidence 0.96) is a reasonable prediction: the post explicitly cites training volume literature, uses the term RIR, and makes a claim framed around research — all strong signals the model learned to associate with the Scientific label.

---

## Reflection: Captured vs. Intended

The model learned the task partially but latched onto surface lexical features rather than the structural intent behind the labels.

What it captured well: **Experiential** has a distinctive voice — first person, informal, often dismissive of "overthinking." The model reliably identifies this register. **Scientific** posts with explicit terminology (RPE, mesocycle, caloric deficit) are also classified confidently and correctly.

What it missed: The **Motivational** label was defined around psychological intent and community role (celebration, etiquette, encouragement), not topic. But the model has no way to detect intent — it sees tokens. A Motivational post that mentions creatine looks like a Scientific post. A Scientific post written in a warm, narrative tone looks like a Motivational post. The model overfit to **topic as a proxy for label** when the actual distinction is **communicative function**. This is a hard problem for a token-level model without context, and fixing it would require either more training data that explicitly shows topic-diverse Motivational examples, or moving to a model with stronger pragmatic understanding.

---

## Spec Reflection

**Where the spec helped:** Writing label definitions and edge case resolution rules in planning.md before annotating a single example was the most valuable step. The hard-edge-case tiebreakers ("if the post's conclusion is psychological, label Motivational even if it mentions a specific lift") made annotation decisions faster and more consistent. Without that spec work, ambiguous posts would have been labeled inconsistently, and the Experiential/Motivational boundary would have been far noisier in the training data.

**Where implementation diverged:** The data collection plan in planning.md called for random sampling from r/beginnerfitness and targeted supplementation if any label fell below 70 examples. In practice, data was collected via Apify with broad queries, and the resulting distribution was close enough to balanced (21 Motivational, 19 Scientific, 21 Experiential in the test set) that no targeted supplementation was needed. The plan also assumed the baseline would perform reasonably above chance — the actual baseline of 34% (close to random for 3 classes) was lower than expected, which made the fine-tuning improvement more dramatic.

---

## AI Usage

**Label stress-testing:** Claude was given the three label definitions and edge case descriptions from planning.md and asked to generate posts that sit at the boundary between Scientific and Experiential, and between Experiential and Motivational. Several generated posts could not be cleanly classified under the original definitions, which led to tightening the tiebreaker rules before annotation began — specifically adding the rule that personal outcome is Experiential regardless of scientific vocabulary used.

**Prompt review:** The initial SYSTEM_PROMPT for the Groq baseline used definitions weaker than planning.md (e.g., "Posts asking for or sharing objective information" for Scientific) and used a question as the Scientific example rather than a declarative claim. Claude identified both issues: the definition failed to capture the periodization/methodology angle, and a question example teaches the model what curiosity looks like, not what Scientific content looks like. The prompt was revised to use the exact planning.md definitions and declarative examples.

**Failure pattern analysis:** After evaluation, all 15 misclassified examples were pasted into Claude with the prompt: "Identify patterns in these misclassified examples — recurring linguistic cues, structural features, or label pairs that keep getting confused." Claude surfaced three patterns: supplement/topic vocabulary pulling Motivational posts toward Scientific, narrative voice pulling Scientific posts toward Motivational, and short low-information posts being unreliable. All three were verified manually against the examples before inclusion in this analysis. One pattern Claude suggested — that post length was a consistent predictor of error — did not hold up on review and was discarded.

---

## Data

- **Source:** r/beginnerfitness, collected via Apify Reddit scraper
- **Total examples:** 348
- **Split:** 243 train / 52 validation / 53 test
- **Labeling:** Manual, using definitions and tiebreaker rules from planning.md
- **Model:** `distilbert-base-uncased`, fine-tuned for 3 epochs
