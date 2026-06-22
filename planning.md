# planning.md — TakeMeter

## Community

I chose r/leagueoflegends, the primary subreddit for the game League of Legends. This community is a strong fit for a classification task because its discourse varies enormously in quality and type — some posts make detailed arguments backed by statistics and mechanics knowledge, others are bold unsubstantiated opinions, and others are pure emotional reactions to events. These distinctions are recognized and discussed by community members themselves (phrases like "actual analysis vs hot take" appear regularly in comment threads), which means the labels are grounded in how the community already thinks about discourse quality.

## Labels

**analysis** — The post makes a structured argument about strategy, champion mechanics, the meta, or statistics. Evidence is specific and verifiable (e.g., winrate numbers, item interaction descriptions, damage calculations). The claim could be evaluated as true or false with game knowledge.

Example 1: "Lethal Tempo on Vayne outperforms Kraken Slayer as the primary damage keystone because her Silver Bolts passive triggers on every auto, meaning attack speed scales her true damage faster than raw AD does past 3 items."

Example 2: "The reason Soraka is strong right now isn't her healing numbers — it's the Moonstone Renewer interaction. With the item's passive triggering off her W heal, she's generating 40+ bonus healing per cast in a teamfight scenario."

**hot_take** — A bold, confident opinion stated without supporting evidence. The claim might be true, but the post asserts rather than argues. Often uses absolute language ("always," "never," "just delete X").

Example 1: "Zed is the most overrated champion in the game. He looks flashy but any decent player can dodge his combos. He's just a noob stomper."

Example 2: "Anyone still playing Ryze at this point is just trolling their team. The champion is fundamentally broken by design."

**reaction** — An immediate emotional response to a specific event, moment, or personal experience. Little to no argument — the post is expressing a feeling. Often uses caps, exclamation points, or personal narrative language.

Example 1: "HOLY MOLY FAKER JUST 1V3ED THE ENTIRE ENEMY TEAM IN THE BARON PIT ARE YOU KIDDING ME"

Example 2: "just got my first pentakill ever with Katarina I am literally shaking my hands won't stop"

## Hard Edge Cases

**The one-stat hot take:** A post that provides a specific statistic but uses it to make an accusatory or emotionally framed claim rather than building a real argument.

Example: "LeBron is overrated — his win rate against top-seeded opponents is below .500."

Decision rule: If the post provides specific verifiable evidence that supports the claim even if you removed the opinion framing, label it **analysis**. If the stat is decorative — cited for credibility but not part of a real argument — label it **hot_take**. A single cherry-picked stat with an accusatory frame = hot_take. Multiple pieces of evidence building a case = analysis.

**The frustrated-but-specific post:** A post that expresses clear frustration but also makes a specific mechanical claim.

Example: "Riot has no idea how to balance mages — they've been broken or useless for 3 seasons straight and the problem is they keep tuning numbers instead of addressing kit overload."

Decision rule: If the frustration is the primary content and the specific claim is incidental, label **hot_take**. If the specific claim is the primary content and the frustration is tone, label **analysis**.

**Difficult annotation cases from the actual dataset:**

1. "The most delusional take I see constantly is that mechanics matter more than game sense. Game sense wins games. Mechanics just make it look pretty." — This sounds like a hot_take but is making a comparative claim that could be argued. Labeled **hot_take** because no evidence is provided and the framing is dismissive rather than analytical.

2. "The ranked experience in this game would be dramatically improved if duo queue was removed entirely from solo queue." — This is an opinion but could be analysis if evidence were provided. Labeled **hot_take** because no supporting reasoning is given.

3. "Season 1 League of Legends was the peak of this game and everything since then has been a slow decline." — Strong opinion with no evidence, but it's not emotionally reactive in the moment. Labeled **hot_take** — it's a stated position, not a reaction.

## Data Collection Plan

Source: Generated synthetic posts modeled on r/leagueoflegends discourse patterns, covering a range of champions, patches, and community topics.

Target: ~67 examples per label (200 total, roughly balanced).

If a label is underrepresented after 200 examples: collect additional examples specifically targeting that label by focusing on the post types that typically generate it (e.g., patch reaction threads for **reaction**, champion discussion threads for **analysis**, ranked complaint threads for **hot_take**).

## Evaluation Metrics

Primary metric: **per-class F1** for each of the three labels. Accuracy alone is insufficient because it hides which specific label boundaries the model fails at. If the model confuses analysis and hot_take but correctly identifies all reactions, overall accuracy looks acceptable while the most important distinction is broken.

Secondary metric: **confusion matrix** to identify directional errors — specifically whether analysis is being misclassified as hot_take or vice versa, which is the hardest boundary.

I will also report **precision and recall** separately for analysis and hot_take, because the cost of a false positive differs from the cost of a false negative in a community tool context. Falsely labeling a hot_take as analysis is worse than the reverse — it would mislead users about the quality of a post.

## Definition of Success

A classifier is useful if:
- Overall accuracy exceeds 75% on the test set
- Per-class F1 for all three labels is above 0.65
- The fine-tuned model meaningfully outperforms the zero-shot baseline (at least 10 percentage points on overall accuracy)

A classifier is not useful if it achieves high accuracy by predicting one label most of the time, or if the analysis/hot_take boundary F1 is below 0.60.

## AI Tool Plan

**Label stress-testing:** I will give Claude my three label definitions and ask it to generate 10 posts that sit at the boundary between analysis and hot_take. If Claude produces posts I can't classify cleanly under my definitions, I will tighten the decision rule before annotating the full dataset.

**Annotation assistance:** I will use Claude to pre-label a batch of 50 examples by providing the label definitions and asking for one label per post. I will review every pre-assigned label and correct any I disagree with. I will track which examples were pre-labeled in my notes and disclose this in the AI usage section.

**Failure analysis:** After generating wrong predictions from the fine-tuned model, I will paste the misclassified examples into Claude and ask it to identify common themes — post length, use of specific language, label pairs that keep being confused. I will then verify those patterns myself by re-reading the examples before writing the evaluation report.
