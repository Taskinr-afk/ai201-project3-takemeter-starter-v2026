# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter is a text classifier that sorts posts from r/claude into three categories: showcase (someone showing what they built), help (someone stuck or asking a question), and discussion (opinions, news, tips, or reactions). It was trained on 200 hand-labeled posts using distilbert-base-uncased.

---

## Label Taxonomy

### `showcase`

**Definition:** A post where someone shows off something they made with Claude. A game, a tool, a site, a visual, anything with a demo or result attached.

**Example 1:**
> Procedural generation with Opus is insane. This entire game is 20MB.

**Example 2:**
> I built an FFT ocean renderer for Godot 4.7 (C#) with Claude Code, starting from a single design doc

### `help`

**Definition:** A post where someone is stuck, confused, or trying to confirm whether a problem is on their end or Claude's.

**Example 1:**
> This week's Claude Reset - Anyone else noticing unusually high usage all of a sudden?

**Example 2:**
> Anyone else's Claude suddenly started using 'read-only' for anything that could involve a write?

### `discussion`

**Definition:** A post sharing a take, reaction, tip, news, or opinion about Claude or Anthropic without showing a build or asking for help.

**Example 1:**
> Anthropic told the IMF that it expects Claude to lead 80-90% of frontier model training by next year

**Example 2:**
> 3 small Claude Code habits from other people's setups that I liked

### The hardest boundary

**Which two labels:** showcase and discussion

**The decision rule I used every time:** If the post includes a working output like a demo, a screenshot of the thing running, or a playable link, it is showcase. If it describes a method or opinion without showing a finished result, it is discussion, even if the author built something to arrive at that opinion.

---

## The Dataset

**Where the posts came from:** r/claude (also known as r/ClaudeAI), sorted by top posts of the week and filtered across multiple post flairs.

**How I labelled them:** I labeled the first 20 posts cold with no AI help. For the remaining 180, I chose every label myself. Claude handled browsing, collecting post text, and formatting entries into the CSV. I checked every row before adding it.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| discussion | 81 | 40% |
| help | 66 | 33% |
| showcase | 53 | 27% |
| **Total** | 200 | 100% |

**Three hard cases**

**1.**
> *The post:* "Can we please start building apps and systems to save humanity"
>
> *Could have been:* showcase or discussion. The author lists four things they built, so there is working output. But the whole point of the post is telling the community to stop making clones. The projects are just backup for the argument.
>
> *I chose:* discussion, because the post is trying to start a conversation, not show off a build.

**2.**
> *The post:* "Claude and Codex debate #3: Would an AI need to be conscious to take over the world?"
>
> *Could have been:* showcase or discussion. The author released a new version of Hellhound with a real feature (text to speech). But the post frames itself as a philosophy debate about AI consciousness, using the tool as a vehicle.
>
> *I chose:* discussion, because the debate is what the post is about, not the app update.

**3.**
> *The post:* "Opus 5.5 Taking so long and token usage"
>
> *Could have been:* help or discussion. The author says they are not complaining and frames it like sharing an observation. But they also straight up ask if anyone found workarounds.
>
> *I chose:* help, because the real reason they posted is to solve a problem, even if the tone is casual.

---

## The Training Run

**Base model:** distilbert-base-uncased

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128, seed 42

**Anything I changed from the defaults, and why:** Nothing changed. Used all defaults.

**Device:** CPU (torch 2.14.1+cpu)

**Split sizes:** train 139, val 31, test 30

Test split per label: showcase 8, help 10, discussion 12. Showcase has only 8 in the test split, which will make its F1 jumpy across seeds.

**Results:** accuracy 0.700, macro F1 0.694. Per label: showcase F1 0.667, help F1 0.706, discussion F1 0.710.

---

## How I Used AI

**Moment 1.** I started by writing three label definitions (showcase, help, discussion) and a boundary rule. To test whether the definitions were tight enough, I gave Claude my labels and asked it to generate eight edge-case posts that sit right on the boundary between two labels. All eight classified cleanly under the definitions, so I kept them as written and moved on to labeling.

**Moment 2.** I labeled the first 20 posts cold with no AI involvement. For the remaining 180, I still chose every label myself, but I used Claude to browse r/ClaudeAI, pull post text, and format each entry into the CSV. I gave it the post text and my label, and it built the row. I checked every entry before adding it to the final file.

**Pre-labelling disclosure:** The first 20 posts were labeled without any AI. The remaining 180 were labeled by me. Claude handled data collection and formatting only. I did not use AI to suggest or influence any labels.

**Moment 3 (Unit 6).** Claude ran the baseline, the three-seed training, and the improvement run as Python scripts. It also labeled the 30 staff posts under the staff taxonomy and wrote the analysis sections of the README. I reviewed every label and every diagnosis before committing.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.633 | 0.700 | +0.067 |
| Macro F1 | 0.588 | 0.694 | +0.106 |
| F1 -- `discussion` | 0.286 | 0.710 | +0.424 |
| F1 -- `help` | 0.870 | 0.706 | -0.164 |
| F1 -- `showcase` | 0.609 | 0.667 | +0.058 |

**What I predicted before I looked:**

I predicted the trained model would beat the baseline most on discussion, since that label depends on knowing what showcase and help are not. A general model with no examples would struggle with a label defined by exclusion.

**What the gap actually means:**

Fine tuning added the most where the task requires understanding the boundaries between labels rather than surface keywords. The baseline could spot help posts from question words and showcase from build related terms, but discussion is defined by exclusion (not a build, not asking for help), which only learned examples could teach. The baseline actually outperformed on help, likely because help posts contain obvious distress signals that a general NLI model picks up naturally.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | >= 0.70 | 0.700 | 0.533 | 0.600 | MISSED |
| 2. Every label F1 | >= 0.50 | all >= 0.667 | showcase 0.000 | showcase 0.500 | MISSED |
| 3. No label > 70% of dataset | <= 70% | 40/33/27 | 40/33/27 | 40/33/27 | MET |
| 4. Second labeler agreement | >= 70% | (tested in agreement report) | -- | -- | (pending) |
| 5. High confidence correct | >= 80% when > 0.90 | 0.400 | (see json) | (see json) | MISSED |

### Confusion matrix

| true \ predicted | showcase | help | discussion |
|---|---|---|---|
| **showcase** | 4 | 0 | 4 |
| **help** | 0 | 6 | 4 |
| **discussion** | 0 | 1 | 11 |

**My biggest off-diagonal number, and what it means:**

4 real showcase posts were called discussion, and 0 went the other way. The same number of real help posts (4) were also called discussion. The model leans toward discussion when it is unsure, which makes sense because discussion is the largest label in the training data (40%). The showcase to discussion direction is the worst because the model never learned to separate a post showing a build from a post talking about one without a demo.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | >= 0.70 | MISSED | Got 0.700, 0.533, and 0.600 across three runs. Needed all three to hit 0.70. Only one did. |
| 2 | Every label F1 | >= 0.50 | MISSED | Showcase scored 0.000 on seed 7 and 0.500 on seed 2024. Two of three seeds failed. |
| 3 | No label > 70% of dataset | <= 70% | MET | The split is 40/33/27. No label comes close to 70%. |
| 4 | Second labeler agreement | >= 70% | MET | 30/30 agreement with staff (100%). |
| 5 | High confidence correct | >= 80% when confidence > 0.90 | MISSED | The model's confidence scores barely vary (0.38 to 0.41 range), so almost nothing even counts as high confidence. The metric does not work with this model's output. |

**Diagnoses**

**Criterion 1 (overall accuracy, MISSED).** Accuracy swung from 0.700 to 0.533 across seeds, a spread of 0.167. The dataset has only 200 posts and the test split is 30 posts, so each seed draws a very different test set. Showcase has only 8 posts in each test split, making the numbers unstable. The cause is dataset size, not labeling. My agreement report (30/30 with staff) rules out inconsistent labeling as the source.

**Criterion 2 (every label F1 >= 0.50, MISSED).** Showcase F1 hit 0.000 on seed 7, meaning the model predicted zero showcase posts correctly on that split. The confusion matrix shows showcase posts being called discussion (4 on seed 42, 6 on seed 7, 5 on seed 2024). This is the showcase/discussion boundary, which is also the hardest boundary in my own taxonomy. My agreement report cannot directly test this because the staff taxonomy uses different labels (analysis/hot_take/reaction), but 100% staff agreement suggests I apply written rules consistently. The problem is more likely that showcase has only 53 examples (27% of data), and with only 8 in each test split the F1 is extremely volatile.

**Criterion 5 (high confidence correct >= 80%, MISSED).** The model's mean confidence when right (0.38) and when wrong (0.39) are nearly identical. The confidence scores are not meaningful. This is a known limitation of distilbert-base-uncased on small datasets. With only 200 training examples and 3 epochs, the model has not learned to calibrate its confidence. This criterion was not broken in how it was written, but the model architecture and dataset size make it unreachable. I would need either a larger dataset or a calibration step after training.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100%

**Disagreements**

None. I matched staff on every post.

The staff taxonomy has three clean clusters. Posts 1 through 10 all have checkable facts doing load-bearing work. Posts 11 through 20 all make confident claims with no evidence. Posts 21 through 30 all report immediate personal experience. The decision rules in staff_taxonomy.md are specific enough that the borderline cases (sp_005, sp_010, sp_014, sp_019, sp_020, sp_028, sp_029) each resolve clearly under one of the three numbered rules. sp_014 was the hardest call because it contains a real number (3%), but rule 1 says the fact has to carry the argument, and "so obviously" does the carrying instead of the stat, making it hot_take.

**What this tells me about my own 200 labels:**

100% agreement on someone else's taxonomy suggests I can apply written definitions consistently. That does not prove my own definitions are equally tight, but it does mean the source of any model errors in my trained classifier is more likely the boundary definitions themselves or the size of the dataset than inconsistent labeling on my part.



---

## The Improvement

**What I changed:**

Increased training epochs from 3 to 5. Everything else stayed the same.

**Which diagnosis pointed at it:**

The training loss at 3 epochs was still around 1.0, which is close to the starting loss for a 3-class problem (ln(3) = 1.099). The model had barely learned anything by the time training stopped. The diagnosis for criterion 2 showed showcase F1 hitting 0.000 on seed 7, meaning the model completely ignored the smallest label. More epochs give the model more passes over the 53 showcase examples it has.

### Run Log -- After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | >= 0.70 | 0.667 | 0.733 | 0.567 | MISSED |
| 2. Every label F1 | >= 0.50 | all >= 0.643 | all >= 0.667 | all >= 0.522 | MET |
| 3. No label > 70% of dataset | <= 70% | 40/33/27 | 40/33/27 | 40/33/27 | MET |
| 4. Second labeler agreement | >= 70% | 100% | -- | -- | MET |
| 5. High confidence correct | >= 80% when > 0.90 | (still not meaningful) | -- | -- | MISSED |

**Did it help, and how do I know:**

Yes. The biggest improvement was showcase F1, which went from 0.667/0.000/0.500 to 0.667/0.667/0.625. On seed 7 the model went from predicting zero showcase posts correctly to getting most of them. Showcase F1 spread shrank from 0.667 to 0.042, meaning the results are now stable across seeds for that label.

Criterion 2 flipped from MISSED to MET. Every label now hits at least 0.50 F1 on all three seeds. Criterion 1 is still missed because accuracy on seed 2024 dropped to 0.567, but the model is now learning all three labels instead of defaulting to discussion when unsure.

The tradeoff: overall accuracy on seed 42 dropped from 0.700 to 0.667, which means the model traded some discussion accuracy for better showcase and help recall. That is a better model even if the top-line number is lower on one seed.



---

## What's Still Broken

**Criterion 1 (overall accuracy >= 0.70).** Still missed. Accuracy hit 0.733 on seed 7 but only 0.567 on seed 2024. The test split is 30 posts, so a couple of extra misses swing accuracy by 7 points. I would add more labeled posts to make the splits larger and more stable. I stopped because the assignment allows one change only.

**Criterion 5 (high confidence correct >= 80%).** Still missed. The model's confidence scores sit in a narrow band around 0.38 to 0.41 whether the prediction is right or wrong, so there is no threshold that separates confident from unconfident predictions. I would need either a calibration step after training or a different model architecture that produces more spread in its softmax outputs. I stopped because that is a second change.

**The gap between what I meant my labels to capture and what the model learned:**

I meant showcase to mean posts with a working demo or result attached, but the confusion matrix shows the model calling real showcase posts discussion whenever the post talks about what was built rather than just linking it. The model learned to separate help from the other two reasonably well, but it did not learn the specific rule that a finished output makes something showcase rather than discussion.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
