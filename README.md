# TakeMeter — r/formula1 Text Classifier
### AI201 Project 3

## Community

r/formula1 was chosen because I follow the sport and am familiar with the community, which helped a lot during annotation and evaluation. The subreddit has a wide range of post types — engineering deep-dives, race recaps, inside jokes, and breaking news — all about the same topic (F1), which makes register-based classification interesting. The subreddit already uses post flairs (Technical, Humour, News, Race Thread), which confirms these are distinctions that matter to the community. A classifier like this could realistically be used to auto-flair new posts or help mods filter by discussion type.

## Label Taxonomy

Four labels based on discussion register:

**technical** — A post focused on the engineering, physics, race strategy, or statistical/historical analysis of F1, either explaining how something works or analyzing performance data.
* "Why does Red Bull's floor generate more downforce than Mercedes at high speed? Is it the venturi tunnel geometry?"
* "Top 4 by win percentage: Clark (34.25%), Verstappen (29.58%), Schumacher (29.55%), Hamilton (27.39%)"

**results_recap** — A post about what happened in a race, qualifying session, or championship standings — including result announcements, summaries, and fan reactions/impressions to those results.
* "Lewis Hamilton wins the 2026 Barcelona Grand Prix, his first victory for Ferrari!"
* "FORZA FERRARI! — reaction to Hamilton's win after a dominant strategy call"

**humor** — A post that is primarily a joke, meme, or lighthearted take — intent is to be funny, not to inform.
* "POV: you're a Haas fan watching Williams accidentally become a midfield contender [crying meme]"
* "Don't cry because it's hornover, smile because it verstappened"

**news_drama** — A post about off-track events — driver transfers, team politics, FIA decisions, or controversies not tied to a specific race result.
* "BREAKING: Christian Horner is to exit Red Bull Racing with immediate effect"
* "FIA changes F1's 2027 engine rules after driver concerns"

## Dataset

**Source:** r/formula1, collected manually by browsing flair filters (Technical, Humour, News, Race Thread) and top posts from the past year. All posts are public.

**Labeling process:** Each post was read individually and assigned one label based on its primary intent. Posts that were genuinely ambiguous between two labels were skipped rather than forced into a category.

**Label distribution:**

| Label | Count |
|---|---|
| technical | 50 |
| results_recap | 50 |
| humor | 50 |
| news_drama | 50 |
| **Total** | **200** |

**Three genuinely difficult annotation cases:**

1. *"Gasly promoted to third place in Monaco GP as stewards strike out penalties"* — between results_recap and news_drama. The content is about a race result change, but triggered by an FIA decision. Labeled **news_drama** because the focus is the stewards action, not the race itself.

2. *"Ferrari upgrades going backwards as usual"* — between humor and technical. Mentions car development, but "as usual" is a well-known community sarcastic meme about Ferrari. Labeled **humor** because the intent is clearly ironic, not informational.

3. *"[Autosport] Putting Lewis Hamilton's F1 career in perspective"* — between technical and news_drama. It is a news article but the content is career statistics and analysis. Labeled **technical** because the substance is performance data, not an off-track event.

## Fine-Tuning

**Base model:** distilbert-base-uncased (HuggingFace)

**Platform:** Google Colab with T4 GPU

The model was fine-tuned twice. The first run used the starter notebook defaults — 3 epochs, learning rate 2e-5, batch size 16 — and got 40% accuracy. Humor was completely unlearned (0/8 correct). The second run increased epochs to 7 and lowered the learning rate to 1e-5 to allow slower, more stable convergence on the small dataset. This improved accuracy to 60% and allowed the model to start predicting humor correctly (4/8). Lowering the learning rate was the more impactful change — at 2e-5 the model appeared to not converge properly on the harder label boundaries with only 140 training examples.

## Baseline

**Model:** Groq llama-3.3-70b-versatile, zero-shot (no fine-tuning, no labeled examples provided).

**Prompt:**

```
You are classifying posts from r/formula1. Assign each post to exactly one category.

technical: A post about F1 engineering, physics, race strategy, or statistical/historical analysis.
results_recap: A post about what happened in a race or qualifying — results, summaries, or fan reactions.
humor: A post that is primarily a joke, meme, or lighthearted take — intent is to be funny, not inform.
news_drama: A post about off-track events — driver transfers, team politics, FIA decisions, or controversies.

Respond with ONLY the label name. Nothing else.
Valid labels: technical, results_recap, humor, news_drama
```

The baseline was run on the same 30-example test set used for fine-tuning evaluation. All 30 responses were parseable with no ambiguous outputs.

## Evaluation Report

### Summary

| Model | Accuracy | Macro F1 |
|---|---|---|
| Fine-tuned DistilBERT | 0.60 | 0.58 |
| Groq baseline (zero-shot) | 0.90 | 0.90 |

Fine-tuning did not beat the baseline. The zero-shot LLM outperformed the fine-tuned model by 30 percentage points on both metrics.

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| technical | 0.70 | 0.88 | 0.78 | 8 |
| results_recap | 0.63 | 0.71 | 0.67 | 7 |
| humor | 0.44 | 0.50 | 0.47 | 8 |
| news_drama | 0.67 | 0.29 | 0.40 | 7 |
| **macro avg** | **0.61** | **0.59** | **0.58** | **30** |

### Per-Class Metrics — Groq Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| technical | 0.89 | 1.00 | 0.94 | 8 |
| results_recap | 1.00 | 0.71 | 0.83 | 7 |
| humor | 0.88 | 0.88 | 0.88 | 8 |
| news_drama | 0.88 | 1.00 | 0.93 | 7 |
| **macro avg** | **0.91** | **0.90** | **0.90** | **30** |

### Confusion Matrix — Fine-Tuned Model

Rows are true labels, columns are predicted labels. Diagonal is correct.

| | pred: technical | pred: results_recap | pred: humor | pred: news_drama |
|---|---|---|---|---|
| true: technical | **7** | 0 | 1 | 0 |
| true: results_recap | 0 | **5** | 2 | 0 |
| true: humor | 1 | 2 | **4** | 1 |
| true: news_drama | 2 | 1 | 2 | **2** |

### Three Wrong Predictions Analyzed

**1. "Peak ragebait from Autosport." — true: humor, predicted: news_drama**

This is a one-liner mocking a media outlet, which r/formula1 uses as a running joke. The model predicted news_drama, probably because "Autosport" is a news source name. There is no linguistic signal that this is a joke — the humor is entirely community-context-dependent. DistilBERT has no way to know "Autosport" is being mocked.

**2. "Hamilton +25 on Antonelli. Walk with me..." — true: results_recap, predicted: humor**

A short post referencing a championship points gap. The clipped, informal phrasing ("Walk with me...") looks like a humor setup structurally. Without understanding it's referencing actual standings data, the model reads it as a punchline rather than a standings update.

**3. "Perez is back. It has to be. I mean, Cadillac is reportedly looking for Perez..." — true: news_drama, predicted: technical**

A speculative driver transfer post. The model likely associated driver and team names with technical/car discussions rather than off-track news. Without understanding the transfer context, this post's vocabulary looks like any other F1 discussion.

### Sample Classifications

Confidence scores for correct predictions are approximate — verify from the Colab notebook's ft_probs output if exact values are needed.

| Post (truncated) | True label | Predicted | Confidence | Correct |
|---|---|---|---|---|
| "Lewis Hamilton wins the 2026 Barcelona Grand Prix..." | results_recap | results_recap | ~0.72 | Yes |
| "Don't cry because it's hornover, smile because it verstappened" | humor | humor | ~0.61 | Yes |
| "Ferrari brought new wheel rims to Barcelona to manage rear tyre overheating..." | technical | technical | ~0.65 | Yes |
| "Peak ragebait from Autosport." | humor | news_drama | 0.27 | No |
| "Hamilton +25 on Antonelli. Walk with me..." | results_recap | humor | 0.28 | No |

The first correct prediction — "Lewis Hamilton wins the 2026 Barcelona Grand Prix, his first victory for Ferrari!" — is a textbook results_recap: it names the driver, the team, the race, and the outcome. Every token is a direct result signal. The model classified it correctly with high confidence.

## Reflection

The model did not meet the success criteria defined in planning.md — macro F1 0.58 fell short of the 0.65 deployment floor, news_drama F1 (0.40) was below the 0.50 class minimum, and the fine-tuned model did not beat the baseline. That said, the fine-tuned model learned something, just not exactly what was intended. For technical and results_recap it does reasonably well (F1 0.78 and 0.67), because those labels have consistent vocabulary — technical posts use engineering terms and results posts use race winner language. The model learned those surface patterns. What it did not learn is register: it cannot distinguish a sarcastic one-liner from a results update if the vocabulary is unfamiliar. Humor in this community is almost entirely based on inside jokes and references ("El plan", "UNC STILL GOT IT") that are meaningless without F1 cultural knowledge, and DistilBERT has none of that. The most surprising outcome was news_drama — it was the best class in run 1 (5/7) but collapsed to the worst in run 2 (2/7) after more training. This suggests the model traded news_drama signal for humor signal as training continued, rather than learning both cleanly. With only 35 training examples per class, the learned boundaries are fragile.

## Spec Reflection

The spec's requirement to run the baseline before fine-tuning was the most useful structural constraint — it forced an honest comparison and made clear early on that the task was easier for a large LLM than for a small fine-tuned model. One way implementation diverged: the spec mentions that manual data collection takes 1–2 hours and is often preferable to building a scraper. I initially tried to automate collection via Reddit's API, ran into authentication issues, and ended up collecting manually anyway. The manual approach ended up being useful because reading 200 posts before labeling gave a much clearer sense of where label boundaries were fuzzy, which improved annotation consistency.

## AI Usage

1. **Planning and label design:** Claude was used to design the planning.md structure and draft the label definitions, example posts, edge case descriptions, and evaluation metrics reasoning. I reviewed all of it and made corrections — the most important one was the results_recap definition, which originally described only summaries and missed fan reactions entirely. I updated it to explicitly include reactions since that's a large part of what those posts actually look like.

2. **Error pattern analysis:** After getting the wrong predictions list from the notebook, Claude was used to identify patterns across the 18 errors. It identified the humor-community-context issue and the news_drama collapse after increased training. I then re-read the specific wrong posts myself to verify those patterns before writing the analysis in this README. The community-context explanation held up; the news_drama explanation was a bit more speculative and I noted it as such.

All 200 labels were assigned manually. No LLM was used for pre-labeling.
