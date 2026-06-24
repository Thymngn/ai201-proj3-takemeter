# Project 3 Planning — Splatoon Sentiment Classification

## Community

**r/splatoon** (Reddit) — the primary fan community for the Splatoon franchise, with ~450k members. Posts range from casual gameplay moments to deep competitive strategy discussions, balance complaints, and community events like Splatfests. This community is ideal for sentiment classification because:

- Discourse is high-volume and varied — multiple posts per hour covering completely different emotional registers
- Players have strong opinions about balance and matchmaking, producing clearly negative posts
- Splatfests and seasonal updates generate consistent positive hype
- Mechanical questions and tips produce a distinct neutral/informational category

The distinctions between positive, negative, and neutral posts are **meaningful to the community** — players actively call out frustration vs. hype, and the community has cultural norms around what counts as a valid complaint vs. a fun moment share.

---

## Label Taxonomy

### `positive`
**Definition:** The post expresses genuine enjoyment, excitement, pride, or appreciation — about a match result, a game mechanic, an update, or the game in general. The emotional tone is clearly favorable.

**Examples:**
- "Just hit S+ for the first time ever!! I can't believe it, been grinding for weeks. This game is incredible."
- "Salmon Run is the most fun I've had in a multiplayer game in years. No cap."

---

### `negative`
**Definition:** The post expresses frustration, criticism, or dissatisfaction — with netcode, balance, matchmaking, Nintendo's decisions, or the community. The dominant emotion is unfavorable.

**Examples:**
- "The netcode in this game is genuinely terrible and Nintendo refuses to acknowledge it."
- "Lost my rank because of a disconnect that wasn't my fault. The penalty system is stupid."

---

### `neutral`
**Definition:** The post is primarily informational — asking a question, seeking tips, sharing a factual observation, or discussing strategy without a strong positive or negative emotional charge.

**Examples:**
- "Does anyone have tips for improving at Splat Zones? I always seem to lose the zone when I try to push."
- "What's the current best weapon for ranked Clam Blitz? Looking to switch mains."

---

## Hard Edge Cases

### Edge Case 1: Frustrated question
"Why does the matchmaking keep putting me against S+ players when I'm B rank? Is this intentional?"

This is a question (→ neutral) but drips with frustration (→ negative). **Decision rule:** If the post's primary communicative act is a question seeking information, label it **neutral** even if the framing is frustrated. If the question is purely rhetorical and the post is really venting, label it **negative**.

**Decision:** neutral — the person is genuinely asking about matchmaking mechanics.

---

### Edge Case 2: Ironic/backhanded positive
"Love how Nintendo still hasn't fixed the disconnect penalty bug after 6 months, truly inspiring commitment to quality."

Sarcasm makes this look positive on the surface (uses "love," "inspiring") but is clearly negative. **Decision rule:** Read for intent, not surface words. If the post is sarcastic criticism, label **negative**.

**Decision:** negative.

---

### Edge Case 3: Neutral with mild positive framing
"Just started playing Splatoon 3 last week. Still learning the basics but enjoying it so far. Any tips?"

Contains mild positivity ("enjoying it") but the primary purpose is to introduce themselves and ask for help. **Decision rule:** If asking for help is the primary communicative act and sentiment is incidental, label **neutral**.

**Decision:** neutral.

---

## Data Collection Plan

**Source:** r/splatoon (Reddit) — publicly accessible posts and comments from Hot, Top (past year), and comment threads on popular posts.

**Target distribution:** ~70 examples per label (positive / negative / neutral) for 210 total examples.

**If a label is underrepresented:** For negative posts specifically, check "controversial" sort. For neutral, check "help" flair and pinned question threads.

**Process:** Posts were read and labeled individually using the definitions above. Ambiguous cases were resolved using the edge case decision rules. A notes column was added for any borderline decision.

---

## Evaluation Metrics

**Primary metric: Per-class F1 score**

Accuracy alone is insufficient because even a balanced 3-class dataset (33% each) would reward a model that learns shortcuts. F1 per class tells us:
- Whether the model handles all three labels or collapses predictions toward one
- Which class boundary is hardest to learn (positive vs. negative is likely easier than neutral vs. positive)

**Secondary metrics:**
- Confusion matrix: to see which label pairs are confused in which direction
- Overall accuracy: for a headline comparison with the baseline

**Success threshold:** Per-class F1 ≥ 0.70 for all three labels on the test set. At that level, the classifier would be usable in a real community moderation or mood-tracking tool.

---

## Definition of Success

A classifier is "good enough for deployment" when:
1. All three per-class F1 scores are ≥ 0.70
2. The fine-tuned model meaningfully outperforms the zero-shot Groq baseline (at least +10% overall accuracy)
3. No single label is predicted for >60% of test examples (avoids majority-class collapse)

---

## AI Tool Plan

### Label stress-testing
Used Claude to generate 5–10 posts that sit at the boundary between **neutral** and **negative** (the hardest boundary in this taxonomy). Reviewed each and refined the decision rule for "frustrated questions."

### Annotation assistance
Labels were assigned directly based on the taxonomy definitions. No LLM pre-labeling was used — all 202 examples were labeled by hand to ensure quality.

### Failure analysis
After fine-tuning, wrong predictions will be pasted into Claude with the prompt: "Identify any common patterns among these misclassified examples — consider post length, sarcasm, question framing, or label confusion direction." Patterns will be verified manually before including in the evaluation report.

---

## Difficult Annotation Decisions (from actual labeling)

**Example 1:**
> "Tower control is frustrating but when you win a close one it is the BEST feeling."

Could be **negative** (frustration) or **positive** (the win feeling). The post ends on a high — the frustration is acknowledged as part of what makes the win satisfying. **→ positive**

**Example 2:**
> "This game has the potential to be the best shooter ever but Nintendo holds it back."

Opens with praise, closes with criticism. Dominant emotional register is **negative** — the praise exists to frame how disappointed the poster is. **→ negative**

**Example 3:**
> "Has the online quality improved since launch? Thinking about coming back."

Mild implied negativity (they left due to quality issues) but the post is a genuine informational question. **→ neutral**
