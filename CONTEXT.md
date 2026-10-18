# CONTEXT.md — read this first every session

## What this project is

Student: Taskin Rahman (GitHub: Taskinr-afk)
Course: AI201 Project 3 — TakeMeter
Goal: build a text classifier that sorts community posts into labels, train it with fine-tuning (distilbert-base-uncased), evaluate it across three seeds, run a zero-shot baseline comparison, and do an agreement check against staff labels.

The project spans two units. Unit 5 is the build (pick a community, design labels, write criteria, collect 200 posts into labels.csv, train). Unit 6 is the test (baseline, three seeds, confusion matrix, agreement report, diagnose and improve).

Key files: takemeter.ipynb (training notebook), baseline.py (zero-shot baseline), agreement.py (staff agreement check), criteria.md (five acceptance criteria), labels.csv (you create this), README.md (the submission).

## Where we are right now

Unit 5 complete. All six milestones done. Community is r/claude. Labels: showcase, help, discussion. 200 posts labeled, training run finished (accuracy 0.700, macro F1 0.694). README filled in. Ready for Unit 6.

## Working agreement

1a. One milestone at a time. Do not jump ahead.
1b. Follow the class slides live. Wait to be told which milestone to start.
1c. Do not push. Do not commit unless Taskin says so.
1d. No Co-Authored-By line in commits. Message format: "Milestone N: two words"
1e. Drafting rule: Claude drafts, Taskin rewrites in own words, Claude implements Taskin's version verbatim. Do not polish Taskin's wording after receiving it.

## Environment notes

Python: 3.11, 3.12, or 3.13 (not 3.14)
Venv: .venv in the project root
Activate (Windows): .venv\Scripts\Activate.ps1
Install: pip install -r requirements.txt
Verify: python test.py
Disk: needs about 9 GB free
RAM: needs about 6 GB

## Writing preferences

2a. Plain text in chat, no markdown bullets.
2b. Number every item like 1a 1b 2a 2b.
2c. Short sentences.
2d. Exact commands and paths.
2e. Say "causes an error" not "raises".
2f. No dashes or colons inside sentences for submitted work.
