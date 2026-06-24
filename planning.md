# Project 3: Text Classification Planning

## 1. Community

**What community did you choose and why?**

<!-- Describe the online community (e.g. a subreddit, forum, Discord, Stack Exchange site, etc.). Explain why you chose it. -->
I'm choosing the F1 community (/formula1). This because I follow the sport and I am familiar with the subreddit, I believe this will help in the data classification as well as evaluation of the project.


**Why is this community a good fit for a classification task?**

<!-- Explain what makes the discourse varied enough to be interesting. What range of post types, tones, or intents exist? Why would a classifier be useful here? -->

The F1 community has a broad range of discussion opportunity, from reactions about recent races and news, to discussions about historical moments or even memes/jokes related to the community. A classiier could take the role of identifing what the discussion is covering other than the topic of F1 (for example, is the post mostly humor/ news or drama reaction/ more technical or historical discussions).

- Posts vary a lot in register: a meme and a tire degradation breakdown look completely different even though both are about F1
- The subreddit already uses flairs (Technical, Humour, News, Race Thread) which confirms these distinctions are meaningful to the community
- A classifier could realistically be used to auto-flair new posts or help mods filter by discussion type

---

## 2. Labels

**What are your 2–4 labels?**

<!-- Define each label in a complete sentence. Then provide 2 example posts per label. -->

Labels were decided to be based on what the discussions are about: technical, results recap, humor, news/drama 

### Label 1: Technical

**Definition:** A post focused on the engineering, physics, race strategy, or historical/statistical analysis of F1 — either explaining how something works, breaking down car concepts, or analyzing driver/team performance data.

**Example post A:** "Why does Red Bull's floor generate more downforce than Mercedes at high speed? Is it the venturi tunnel geometry?"

**Example post B:** "Top 4 by win percentage: Clark (34.25%), Verstappen (29.58%), Schumacher (29.55%), Hamilton (27.39%) — numbers from statsf1.com"

---

### Label 2: Results Recap

**Definition:** A post about what happened in a race, qualifying session, or championship standings — including announcements of results, summaries of events, and fan reactions/impressions to those results.

**Example post A:** "Lewis Hamilton wins the 2026 Barcelona Grand Prix, his first victory for Ferrari!"

**Example post B:** "FORZA FERRARI! — reaction to Hamilton's win after a dominant strategy call"

---

### Label 3: Humor 

**Definition:** A post that is primarily lighthearted, ironic, or meme-based — the intent is to be funny rather than to inform or discuss seriously.

**Example post A:** "POV: you're a Haas fan watching Williams accidentally become a midfield contender [crying meme]"

**Example post B:** "Fernando Alonso's helmet design is just a picture of him staring at a calendar waiting for a competitive car."

---

### Label 4: News/Drama 

**Definition:** A post reporting or reacting to off-track events — driver transfers, team politics, FIA decisions, penalties, or controversies not tied to a specific race result.

**Example post A:** "Breaking: Hamilton confirmed at Ferrari for 2025. Official statement just dropped."

**Example post B:** "Verstappen's penalty is confirmed — drops to P4. Red Bull filing an appeal tonight."

---

## 3. Hard Edge Cases

**What type of post will be genuinely ambiguous between two labels?**

<!-- Describe the specific ambiguity — which two labels does it fall between, and what makes it hard to resolve? -->

A post reacting to a controversial race penalty (e.g. "that 5-second penalty cost Verstappen the win and it was completely wrong") sits between **News/Drama** and **Results Recap** — it's recapping what happened but the primary energy is outrage about a decision, not a neutral summary.

**Which label pair is most likely to cause disagreement?**

News/Drama vs Results Recap — especially post-race drama posts. Second-hardest: Humor vs News/Drama, since sarcastic/ironic posts about real controversies could read as either.

**How will you handle ambiguous posts during annotation?**

<!-- Describe your resolution strategy: a decision rule, a tie-breaking convention, skipping the post, assigning a soft label, etc. -->

Priority rule based on what the **title** signals (since most people only read the title):
- Title sounds like a summary of events → Results Recap
- Title focuses on a decision, controversy, or off-track event → News/Drama
- Title is clearly a joke/meme → Humor
- If still 50/50 after applying the rule → skip the post rather than force a label

**Three specific examples that were genuinely difficult during annotation:**

1. *"Gasly promoted to third place in Monaco GP as stewards strike out penalties"* — sits between results_recap and news_drama. The content is about a post-race result change, but caused by an FIA stewards decision. Labeled **news_drama** because the focus is the stewards action (off-track decision), not the race itself.

2. *"Ferrari upgrades going backwards as usual"* — sits between humor and technical. Mentions car development, but "as usual" is a well-known sarcastic community meme. Labeled **humor** because the intent is clearly ironic, not informational — anyone familiar with the subreddit reads this as a joke.

3. *"[Autosport] Putting Lewis Hamilton's F1 career in perspective"* — sits between technical and news_drama. It's a news article link but the content is statistical/career analysis. Labeled **technical** because the substance is performance data, not an off-track event.

---

## 4. Data Collection Plan

**Where will you collect examples?**

<!-- Name the specific source(s): subreddit name, API, scraper, manual copy-paste, existing dataset, etc. -->

r/formula1, manual copy-paste, browsing by flair (Technical, Humour, News, Race Thread) and top posts of the past year.

**How many examples per label?**

| Label | Target count |
|-------|-------------|
| Technical | 50 |
| Results Recap | 50 |
| Humor | 50 |
| News/Drama | 50 |
| **Total** | **200** |

**What will you do if a label is underrepresented after 200 examples?**

<!-- Describe your fallback: different search terms, different time window, a different subforum, relaxing the definition slightly, etc. -->

- **Technical underrepresented** → check r/f1technical or search "explain" / "how does" in r/formula1
- **Results Recap underrepresented** → pull from weekly race megathread comment sections
- **Humor underrepresented** → search "meme" in post titles or sort by top posts in the Humour flair
- **News/Drama underrepresented** → look at posts from high-drama periods (e.g. Horner investigation, Hamilton-Ferrari news cycle)

---

## 5. Evaluation Metrics

**Which metrics will you use to evaluate your model?**

<!-- Choose from: accuracy, precision, recall, F1 (macro/micro/weighted), AUC-ROC, confusion matrix, etc. -->

- Macro F1 (primary metric)
- Per-class precision and recall
- Confusion matrix

**Why are those the right metrics for this specific task?**

<!-- Accuracy alone is not sufficient. Explain the class balance situation and the relative cost of false positives vs. false negatives for each label. Connect those costs to which metric you choose. -->

Classes should be roughly balanced (50 each), so accuracy isn't terrible here but it still hides which labels are failing. Macro F1 averages F1 across all 4 classes equally, so a model that nails 3 labels but completely fails on one still gets a low score. Per-class breakdown shows exactly where the confusion is. The confusion matrix shows the *direction* of errors (e.g. Technical being predicted as Results Recap vs. being predicted as Humor tells a very different story).

**Are any errors worse than others? Why?**

<!-- For example: is a false negative on a harmful-content label worse than a false positive? -->

No label is safety-critical, so no error is catastrophically worse than another. That said, confusing Technical with Results Recap would suggest the model isn't learning register at all (both are informational), which is a more interesting failure than confusing Humor with Technical (which is much more linguistically distinct).

---

## 6. Definition of Success

**What performance would make this classifier genuinely useful?**

<!-- Describe the threshold in concrete metric terms (e.g., "macro F1 ≥ 0.75") and the use case it unlocks. -->

Macro F1 ≥ 0.75 across all 4 labels — at that point the classifier could reliably auto-flair new posts with only occasional mod correction needed.

**What would you accept as "good enough" for deployment in a real community tool?**

<!-- Be specific about the floor — what is the minimum acceptable performance before you'd consider shipping it, and what safeguards (human review, confidence threshold, etc.) would you add? -->

Macro F1 ≥ 0.65 with a confidence threshold: only auto-flair when the model's confidence is > 0.80, route everything else to human review. That way low-confidence predictions don't pollute the flair system.

**What does failure look like?**

<!-- Describe a metric result or error pattern that would tell you the model is not ready, or that the task design needs to change. -->

- Any single class F1 < 0.50 → that label boundary isn't learnable from 200 examples, needs redesign
- Fine-tuned model doesn't beat the Groq zero-shot baseline → training data is too noisy or labels are inconsistent
- Model predicts one label for >60% of test examples → class imbalance wasn't caught during collection

**Post-implementation note:** The final fine-tuned model (macro F1 0.58) did not meet the "good enough" threshold of ≥ 0.65, and two of the three failure conditions were triggered — news_drama F1 was 0.40 (below the 0.50 floor) and the fine-tuned model did not beat the zero-shot baseline. The success criteria were the right ones to set; the model just didn't reach them with 140 training examples and community-specific humor.

---

## 7. AI Tool Plan

**Label stress-testing**

Before annotating 200 examples, I'll paste my 4 label definitions into Claude and ask it to generate 10 posts that sit on the boundary between two labels (especially News/Drama vs Results Recap and Humor vs News/Drama). If I can't cleanly classify the generated posts myself, I'll tighten the definitions before starting annotation.

**Annotation assistance**

I'll collect all 200 posts first, then potentially use an LLM to pre-label a batch using my definitions. If I do this, I'll add a `pre_labeled` column to the CSV to track which ones were AI-assisted, and I'll review and correct every single pre-assigned label before saving the final file.

**Failure analysis**

After running the notebook and getting the list of wrong predictions, I'll paste them into Claude and ask it to identify common patterns (e.g. short posts, sarcasm, posts where topic signals one label but tone signals another). I'll verify any patterns it finds by re-reading the examples myself before writing them up in the README.
