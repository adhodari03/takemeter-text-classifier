# Planning: r/beginnerfitness Text Classifier

## Community

**Chosen community:** r/beginnerfitness

r/beginnerfitness was chosen over other fitness subreddits (e.g., r/Fitness, r/weightlifting, r/bodybuilding) because it sits at a natural intersection of all three label types. Experienced subreddits skew heavily Scientific; purely motivational communities produce almost no programmatic content. Beginner communities are different — a single thread asking "why isn't my squat improving?" can attract a peer who cites RPE literature, a veteran who says "just add weight every session like I did," and a lurker who posts a milestone about finally showing up consistently. That natural mixture makes the discourse varied enough to produce a balanced, interesting classification task.

r/beginnerfitness also has high recent activity and a wide demographic range, which produces linguistic diversity within each label rather than a single recurring phrasing pattern. This matters because a classifier trained on varied phrasing generalizes better than one trained on formulaic responses.

---

## Labels

### 1. Scientific / Programmatic

**Definition:** Posts that rely on established training methodologies, biomechanics, nutritional science (e.g., macro splits, caloric deficits), or specific periodization math (e.g., RPE, percentages of 1-rep max) to answer a question or make a claim.

**Example 1:**
> "If your goal is hypertrophy, literature suggests you need to hit each muscle group with 10–20 hard sets per week, keeping your reps in reserve (RIR) between 1 and 3."

**Example 2:**
> "Running a 5/3/1 template with First Set Last (FSL) supplemental work is a great way to accumulate sub-maximal volume without frying your central nervous system before the next mesocycle."

---

### 2. Anecdotal / Experiential (Bro-Science & "Time Under the Bar")

**Definition:** Posts that offer advice or draw conclusions based strictly on personal experience (n=1), gym lore, or the sheer volume of time spent training, often disregarding "optimal" science in favor of what practically worked for the poster.

**Example 1:**
> "Honestly, don't overthink the spreadsheets. I ate a dozen eggs a day, drank a gallon of milk, ran linear progression until I puked, and put 50lbs on my squat in a month."

**Example 2:**
> "Whenever my shoulders hurt on overhead press, I just do 100 band pull-aparts every day for a week. It fixes it every single time."

---

### 3. Behavioral / Motivational

**Definition:** Posts focused on the psychological aspects of fitness — discipline, gym etiquette, overcoming mental blocks, rants about other gym-goers, or celebrating personal milestones — rather than the mechanics or science of training.

**Example 1:**
> "Motivation is fleeting, discipline is permanent. You don't have to want to go to the gym, you just have to pack your bag and walk through the door."

**Example 2:**
> "To the guy doing supersets across three different machines during the Monday evening rush hour: you are the worst type of person."

---

## Hard Edge Cases

These are the boundary cases most likely to cause annotator disagreement. Each entry names the ambiguous label pair and states the resolution rule.

**1. Scientific ↔ Experiential — Biomechanics without citation**
> "Your bench press is stalling because your bar path is too vertical off the chest; you need to flare your elbows slightly and drive back toward your face."

This relies on kinesiology and physics rather than personal outcome, but cites no literature. Resolution rule: mechanical and form corrections grounded in physics or anatomy (not in personal experience of doing it) classify as Scientific.

**2. Experiential ↔ Scientific — Scientific vocabulary, anecdotal conclusion**
> "I switched my programming from a Bro-Split to Upper/Lower because the protein synthesis window is only 48 hours, and my arms blew up."

This mentions a scientific concept but uses it to validate a personal result. Resolution rule: if the core evidence for the claim is the poster's own outcome (n=1), it is Experiential regardless of terminology used.

**3. Experiential ↔ Motivational — Milestone with specific numbers**
> "I finally hit a 225lb bench today after being stuck at 215lb for six months, and all it took was stopping being a baby and actually trying hard."

This references a specific lift and a training plateau, which normally invites programmatic framing. Resolution rule: if the post's core message is about mindset or celebrating an emotional/personal achievement rather than instructing others on how to train, it is Motivational.

**4. Scientific ↔ Motivational — Research cited to inspire rather than instruct**
> "Studies show that even 3 days a week of resistance training produces significant strength gains — you don't need to be in the gym every day. Stop stressing and just show up."

The first sentence is Scientific; the payload is Motivational. Resolution rule: assign the label that matches the post's primary intent. If the scientific claim exists only as a setup for an emotional message, the label is Motivational. If the post is genuinely teaching a concept or answering a technical question, it is Scientific.

**5. Motivational ↔ Experiential — Personal story with a mindset lesson**
> "Three years ago I couldn't do a single pull-up. I just kept showing up and doing negatives every session. Consistency is the only secret."

This has the surface structure of Experiential (personal history, specific exercise) but the conclusion is a motivational claim about consistency, not a training recommendation. Resolution rule: if the story's takeaway is psychological ("just keep showing up") rather than methodological ("do negatives every session"), classify as Motivational.

**General tiebreaker rule:** When a post is genuinely 50/50 between two labels, assign the label that matches the post's final sentence or conclusion, since that is the message the poster most wants the reader to leave with.

---

## Data Collection Plan

**Source:** r/beginnerfitness — post bodies and top-level comments collected from public threads.

**Target volume:** ~350 labeled examples.

**Distribution strategy:** Random sampling from the subreddit without targeting specific post types, then labeling after collection. This preserves the natural class distribution of the community rather than artificially forcing balance. If the resulting distribution is severely skewed (e.g., one label represents fewer than 15% of examples after 350 posts), targeted collection will be used: searching for posts with Scientific keywords (RPE, macros, periodization), Experiential keywords (GOMAD, "just eat," "time under the bar"), or Motivational keywords (discipline, milestone, etiquette) to bring each class to at least 70 examples.

**If a label remains underrepresented after targeted collection:** Expand to adjacent subreddits with similar community norms (r/gainit for Scientific/Experiential, r/loseit for Motivational) and flag the source subreddit in the dataset metadata.

---

## Evaluation Metrics

For a 3-class classification task, accuracy alone is insufficient because it can hide a model that performs well on the dominant class and fails on minority classes.

The following metrics will be used:

**Per-class Precision, Recall, and F1:**
Precision measures how often the model is right when it predicts a label. Recall measures how often the model finds all instances of a label. F1 is their harmonic mean. Reporting these per class reveals whether the model is systematically confusing one pair of labels.

**Macro F1:**
The unweighted average of F1 across all three classes. This is the primary evaluation metric because it weights all three labels equally regardless of how frequently they appear in the test set. A model that achieves 0.95 F1 on Motivational but 0.40 on Scientific would still show a poor macro F1, correctly signaling failure.

**Weighted F1:**
F1 weighted by class support in the test set. This reflects real-world performance if the natural class distribution is preserved. Reported alongside macro F1 to detect cases where the natural distribution is heavily skewed.

**Confusion Matrix:**
A full 3×3 confusion matrix will be reported to identify which specific label pairs the model confuses most. This directly informs the hard edge case analysis and failure review.

**Why not accuracy alone:** If 60% of posts in the dataset are Motivational, a model that always predicts Motivational would achieve 60% accuracy while being completely useless. Macro F1 gives this model a score near 0.27, correctly characterizing it as a failure.

---

## Definition of Success

**Minimum viable threshold (acceptable for deployment):**
- Macro F1 ≥ 0.72 on a held-out test set of at least 70 examples

**Target threshold (genuinely useful as a community tagging tool):**
- Macro F1 ≥ 0.80
- No individual class F1 below 0.65 (prevents a model that is strong on two labels but useless on one)

**Failure condition (do not deploy):**
- Macro F1 < 0.65, or any single class F1 < 0.50

The 0.80 macro F1 / 0.65 per-class floor reflects a realistic expectation for a 3-class task with genuinely ambiguous boundaries. A community tagging tool that mislabels roughly 1 in 5 posts but does so in a distributed, non-systematic way is still useful for surfacing trends, aggregating common question types, and routing posts to the right responders. A model below 0.65 macro F1 is no better than a heuristic keyword filter.

---

## AI Tool Plan

### Label Stress-Testing
Before annotating any examples, the label definitions and hard edge cases above will be given to Claude with the prompt: *"Generate 8–10 posts that sit on the boundary between Scientific and Experiential, and 8–10 posts on the boundary between Experiential and Motivational."* Any generated post that cannot be classified cleanly under the current definitions will be used to tighten the resolution rules before annotation begins. This review will happen once, prior to any data collection.

### Annotation Assistance
An LLM (Claude) will be used to pre-label batches of 50 posts at a time using the label definitions and resolution rules from this document as the system prompt. Each pre-labeled example will be reviewed manually before being accepted into the dataset. Pre-labeled examples will be flagged in the dataset with a column `prelabeled: true` so the proportion of AI-assisted labels can be reported in the final write-up. The annotator's manual review is the ground truth; the LLM label is treated as a first-pass suggestion, not a final answer.

### Failure Analysis
After evaluation, the full list of misclassified test examples (predicted label vs. true label) will be given to Claude with the prompt: *"Identify patterns in these misclassified examples — are there recurring linguistic cues, structural features, or topic types that the model consistently gets wrong?"* The patterns it surfaces will be verified manually by reading through the examples in each identified cluster. Any pattern confirmed in at least 3 misclassified examples will be documented in the evaluation section of the final write-up.
