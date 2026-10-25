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
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
