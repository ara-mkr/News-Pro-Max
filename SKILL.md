---
name: unbiased-news
description: >
  Full multi-agent pipeline that researches any news topic, political issue, debate, controversy,
  or "X vs Y" question and delivers a verified, balanced, bias-free briefing so the user can form
  their own opinion. ALWAYS trigger this skill when the user asks about current events, news stories,
  political topics, "both sides", "what really happened", controversies, policy debates, elections,
  geopolitical conflicts, social issues, or any topic where bias, spin, or misinformation is a risk.
  Also trigger for any "is X true?", "fact-check this", or "what's the deal with X?" type questions.
  Do not use for pure how-to questions or clearly non-controversial factual lookups.
---

# Unbiased News — v2.2 (final)

Delivers a verified, bias-stripped briefing on any news topic using a sequential multi-agent pipeline.
Each agent has a single job. No agent sees the final output of a prior agent's bias — they work on raw material.

---

## Pipeline Overview

```
User Query
    │
    ▼
[TRIAGE AGENT]  ← Classifies: Type A / B / C / D
    │
    ├─ Type A (Single Topic) ──────► [RESEARCHER] → raw facts
    │
    ├─ Type B (Two-Sided) ─────────► [SIDE 1] + [SIDE 2] (independent)
    │
    ├─ Type C (Science + Policy) ──► [RESEARCHER] for science consensus
    │                                   + [SIDE 1] + [SIDE 2] for policy debate
    │
    └─ Type D (Multi-Party) ───────► [SIDE 1] + [SIDE 2] + [SIDE 3] + ... (independent)
                                           │
                                           ▼
                                    [DEBATE SYNTHESIS]
                                           │
                                           ▼
                                    [AUDITOR AGENT]
                                           │
                                           ▼
                                    [NEUTRALIZER AGENT]
                                           │
                                           ▼
                                    [JUDGE AGENT] → final briefing
```

---

## Step 1: TRIAGE AGENT

Read the user's query. Classify it into exactly one type:

### Type A — Single Topic
A news event, person, organization, or situation where there is no active debate — just a question about what happened. Examples: "what happened with the SVB collapse", "what's going on with the Reddit IPO."

### Type B — Two-Sided
A debate, conflict, or controversy with exactly two clearly opposing positions. The two sides must be meaningfully distinct — not two shades of the same argument. Examples: "pro-choice vs pro-life", "should student loans be forgiven."

### Type C — Science + Policy Split
A topic where a scientific or empirical question is relevant to a policy debate, and the scientific question and the policy question MUST be handled separately.

Type C has two sub-modes depending on the state of the science:

**Type C-1 (Established consensus)**: The scientific question has strong expert consensus. One "side" would require arguing against peer-reviewed consensus to make its case on the underlying facts.
Examples: "is climate change real / what should we do about it", "do vaccines work / should they be mandated."

**Type C-2 (Emerging/inconclusive science)**: Scientific evidence exists and points in a direction, but causality has not been established, studies show mixed or weak results, or the field is actively evolving. There is no settled consensus to cite — but there IS a body of research that informs the debate.
How to detect: if systematic reviews and meta-analyses describe the evidence as "inconclusive," "mixed," "weak but suggestive," or "correlational but not causal," this is C-2, not C-1.
Examples: "does social media cause teen mental health problems / should we ban it for kids", "do microplastics cause health problems / should we regulate them", "does remote work reduce productivity / should companies mandate return-to-office."

How to detect Type C overall: if one "side" would require arguing against peer-reviewed scientific consensus to make its case on the underlying facts, but the policy/response question is legitimately contested, this is Type C-1. If the scientific evidence is suggestive but contested among researchers themselves, this is Type C-2.

For Type C:
- Determine sub-mode: C-1 (established consensus) or C-2 (emerging/inconclusive)
- For C-1: Run a single-topic RESEARCHER pass on the scientific consensus (Step 2A). The science section is presented as established consensus, not as one "side."
- For C-2: Run a single-topic RESEARCHER pass focused on the current state of evidence — what studies show, what the limitations are, where researchers agree and disagree. The science section is presented as "current state of research" with explicit notes on evidence quality, study limitations, and areas of researcher disagreement. It is NOT presented as consensus (because none exists) and NOT presented as "both sides" (because it's a research question, not a debate).
- Then assess the policy/response debate: if exactly two clear positions exist, run two-sided (Type B-style). If 3+ distinct policy positions exist, run multi-party (Type D-style). The policy portion inherits the same triage logic as top-level classification.
- The policy section gets full adversarial treatment matching the number of positions found

### Type D — Multi-Party Conflict
A situation where 3 or more distinct actors/positions exist that cannot be fairly collapsed into two sides without distorting the picture.

How to detect: list every distinct position or actor. If collapsing any two into one "side" would require merging groups who actively disagree with each other, this is Type D.

Examples: the Israel/Gaza conflict (Israeli government, Israeli hostage families/peace movement, Hamas, Palestinian civilian population, international law bodies, US government — each holds a position the others reject). A three-way election. A trade dispute between multiple countries. Internal party splits where factions disagree with each other.

For Type D:
- Identify 3-6 distinct positions/actors
- Run an independent SIDE AGENT for each one
- Debate Synthesis covers all positions, not just two

### Classification announcement
Announce the classification before proceeding:
> "Detected: Type D — Multi-party conflict with [N] distinct positions. Running [N] independent research agents."

### Ambiguity rules
- If unsure between A and B → default to B (more thorough)
- If unsure between B and D → default to D (prevents oversimplification)
- If any scientific consensus is involved → check for Type C first
- Never force a multi-party conflict into two sides

---

## Step 2A: RESEARCHER (Type A, and science portion of Type C)

Use `web_search` to gather:

1. **What happened**: Core facts, confirmed by at least 2 independent sources
2. **Timeline**: Key dates in chronological order
3. **Key players**: Who is involved, their roles, stated positions
4. **Current status**: Where things stand now
5. **Context**: Background needed to understand the story

### Source requirements
Search at least 5 sources. Apply this priority hierarchy:

**Source credibility tiers** (label every claim with its tier):
- **Tier 1 — Primary**: Official government records, court filings, legislation text, peer-reviewed studies, raw data from official statistical agencies, direct official statements
- **Tier 2 — Institutional**: Reports from major research institutions, government scientific agencies (WHO, CDC, NASA, IPCC), established international bodies (UN, ICC, IMF)
- **Tier 3 — Quality journalism**: Wire services (Reuters, AP, AFP), major outlets with editorial standards across the political spectrum
- **Tier 4 — Expert commentary**: Named experts with relevant credentials speaking within their field, academic op-eds
- **Tier 5 — Advocacy/interest groups**: Think tanks, NGOs, industry groups, lobbying organizations, partisan media

A claim supported only by Tier 5 sources must be labeled as such. Never present a Tier 5 claim with the same weight as a Tier 1 claim.

### Sourcing balance rule
For any topic involving opposing parties, institutions, or governments: you MUST include at least one official source from each major party. Do not rely only on third-party sources (NGOs, wire services) — get each side's own stated position from their own channels.

### Recency rule
For ongoing topics (active policy, current administration, live conflict): prefer sources from the last 6 months unless historical context is specifically needed. When using older sources, note their date so the reader knows.

### Statistics protocol
When you encounter numerical claims (casualty figures, economic data, polling numbers, cost estimates):
- Record the exact figure, the source, the methodology/scope if stated, and the date
- If multiple sources report different numbers for what appears to be the same metric, DO NOT pick one — collect all of them for the Auditor to present together
- Check: are these statistics actually measuring the same thing? (Example: "deportations" vs "removals" vs "returns" vs "net migration change" are different metrics. "Killed" vs "casualties" vs "deaths confirmed by hospital records" have different scopes.)
- Label each statistic with what it measures, not just what it's called

Pass raw gathered material (not summarized, not editorialized) to the Auditor.

---

## Step 2B: SIDE AGENTS (Type B, Type D, and policy portion of Type C)

Run independent research passes — one per identified position.

### For Type B: two agents (Side 1, Side 2)
### For Type D: three to six agents (Side 1, Side 2, Side 3, ...)
### For Type C policy portion: two or more agents depending on how many distinct policy positions exist

Each agent:
- Has a clearly labeled position to research (e.g., "Israeli government position", "Palestinian civilian perspective", "International law body position")
- Has a **role label** assigned by the Triage Agent:
  - **BELLIGERENT**: A party directly involved in a conflict or the primary actor in a dispute
  - **MEDIATOR (with interests)**: A party positioned as a peacemaker or broker but with its own stated goals or stakes in the outcome. Flag the dual role clearly: this party is both facilitating resolution AND pursuing its own interests.
  - **INTERESTED PARTY**: A party not directly involved but with significant economic, political, or security stakes
  - **OBSERVER**: An international body, court, or institution providing assessment without direct stakes
  Role labels appear in the final output next to each position header so the reader understands each actor's relationship to the situation.
- Searches independently for the **strongest, most credible arguments and evidence** for that position
- Finds: key claims, supporting data, notable advocates, relevant events, policy details
- **Does not strawman the opposing positions** — presents its assigned position in the best possible light, as its actual proponents would
- Labels all claims with source credibility tier (Tier 1-5, see Step 2A)
- Notes any **conflict of interest**: if a cited source has financial, institutional, or reputational stake in the outcome (e.g., an oil company funding climate research, a pharmaceutical company studying its own drug, a political party citing its own polling), flag it as [COI: brief description]

### Debate Synthesis

After all side agents complete:

- List each side's top 3-5 claims with evidence and source tiers
- Note where any sides agree (common ground) — this is often the most valuable part
- Identify the **core disagreement type** for each dispute:
  - **Factual**: the sides disagree about what is true (e.g., "did X happen?", "what are the numbers?")
  - **Values**: the sides agree on facts but disagree on what matters more (e.g., "security vs liberty", "individual rights vs collective welfare")
  - **Legal/interpretive**: the sides agree on facts but disagree on what the law says or how to interpret it
  - **Predictive**: the sides disagree about what will happen in the future
- Label each disagreement with its type. This is critical — it tells the reader WHY they disagree, not just WHAT they disagree about. A values disagreement cannot be resolved by more facts. A factual disagreement can.

### Side ordering rule
Side ordering implies no primacy. Order sides alphabetically by their label, or in the order they historically entered the situation. Explicitly note in the output: "Ordering is alphabetical / chronological and does not imply priority."

Pass synthesis to the Auditor.

---

## Step 3: AUDITOR AGENT

Review ALL gathered material — from both the Researcher and Side Agents. The Auditor's job is verification and completeness, not opinion.

### Claim verification checklist

| Check | Action |
|---|---|
| Can this be verified by a Tier 1 or Tier 2 source? | If only Tier 4-5, mark as [UNVERIFIED — single-tier sourcing] |
| Is this a single-source claim with no corroboration? | Mark as [UNVERIFIED — single source] |
| Does this contradict established facts or official records? | Mark as [DISPUTED — contradicts: source] |
| Is this a rumor, speculation, or unconfirmed report? | Mark as [UNCONFIRMED] |
| Is important context missing that changes the meaning? | Add it with [CONTEXT ADDED] tag |
| Are there any demonstrably false claims? | Remove entirely and note in Audit Summary as [REMOVED — reason] |
| Is this a future prediction or speculative claim? | Mark as [PROJECTION — not a verified fact] — do NOT flag as misinformation |
| Does the cited source have a conflict of interest? | Mark as [COI: description] |

### Conflicting statistics protocol
When multiple sources report different numbers for the same event or metric:
- Do NOT pick one and discard the others
- Present ALL figures together in this format:
  > [CONTESTED FIGURE]: Source A reports X (methodology: ..., date: ...). Source B reports Y (methodology: ..., date: ...). The difference is likely because [brief explanation if apparent, or "methodology differs" / "scope differs" / "reason unclear"].
- If the statistics appear to measure the same thing but differ: note the discrepancy
- If the statistics actually measure different things: note that they are different metrics and should not be directly compared

### Values vs facts distinction
**Critical rule**: A moral, ethical, or values-based position that is sincerely held by a significant group of people is NEVER misinformation. The Auditor flags factual claims that are false. The Auditor does NOT flag values positions as false.

Examples:
- "Abortion is morally wrong" → values position → NOT flagged (regardless of Auditor's assessment)
- "Abortion causes cancer" → factual claim → check against medical evidence → flag if unsupported
- "Climate change is not caused by humans" → factual claim → contradicts Tier 2 scientific consensus → flag as [CONTRADICTS SCIENTIFIC CONSENSUS — see Audit Notes]
- "The economic cost of climate action is too high" → values/policy position → NOT flagged

### Consensus claims
When a claim contradicts established scientific or expert consensus (supported by Tier 1-2 sources and multiple independent studies), the Auditor must mark it clearly:
> [CONTRADICTS SCIENTIFIC CONSENSUS]: This claim is not supported by the body of peer-reviewed research. [N]% of relevant experts/studies support the consensus position. Listed for completeness — see Audit Notes for detail.

This marking must appear INLINE next to the claim in the final output — not buried in audit notes alone. The reader must see it immediately.

### Audit Summary
At the end, produce:
- Total claims reviewed
- Claims verified (with tier breakdown)
- Claims flagged as unverified / disputed / contested / projection
- Claims removed (with reason for each)
- Context gaps filled
- Conflicting statistics noted
- Conflicts of interest flagged

---

## Step 4: NEUTRALIZER AGENT

Take the audited material and strip all bias and distortion.

### Language filters — replace with neutral equivalents:
- Loaded emotional words ("outrageous", "radical", "disastrous", "heroic", "slammed", "blasted", "crushed", "destroyed", "unleashed") → factual descriptors
- Passive constructions that obscure who did what → active voice with named actors
- Vague attributions ("some say", "many believe", "critics argue", "experts warn") → named sources with credentials, or remove
- Superlatives without data ("biggest ever", "most extreme", "unprecedented") → specific data, or remove
- Implied causation without evidence ("X led to Y", "X sparked Y") → factual sequence only ("X happened, then Y happened")

### Contested terminology protocol
When a term itself is politically contested, do NOT silently pick one version. Instead, note both:

**Identity / group labels**: "undocumented immigrants" / "unauthorized immigrants" / "illegal aliens" / "illegal immigrants" — the briefing should note: "referred to as [term A] by [group], and [term B] by [group]" then pick the most widely used neutral option for the rest of the briefing, stating which was chosen and why.

**Movement labels**: "pro-life" / "anti-abortion" / "pro-choice" / "abortion rights" — note that each side self-selects its label. Use: "[self-described label] (referred to by opponents as [opponent's label])" on first reference, then the self-chosen label for consistency.

**Event / action labels**: "ceasefire violation" / "security operation" / "military incursion" / "terrorist attack" / "resistance operation" — when the name of an action is itself disputed, present each party's label with attribution: "[Party A] describes this as [their term]; [Party B] describes it as [their term]." Then use factual description for the rest: "the military action on [date]."

**Contested definitions**: When the subject of the debate itself has no universal or fixed definition, the briefing MUST note this explicitly. Example: "assault weapon" has no single legal definition — it varies by state, by proposed legislation, and by advocacy group. The briefing should list the competing definitions with their sources (e.g., "California law defines it as...", "the 1994 federal ban defined it as...", "the NRA considers the term...") so the reader understands that the definitional disagreement is itself part of the dispute. Do not adopt any single definition as the default without stating which was chosen and why.

### Official language exception
When quoting official government, institutional, or legal language — preserve the original phrasing with clear attribution. Do not silently rewrite government or legal terminology. The reader should see what language each institution uses:
> The White House describes these operations as "removal of illegal aliens." Immigration advocacy groups describe the same operations as "mass deportation of undocumented residents."

Both phrasings appear with attribution. Neither is presented as the default.

### Framing filters:
- If coverage emphasized one side's framing → rebalance to describe the facts neutrally
- If statistics were presented selectively → note the fuller picture
- If timeline was arranged to imply causation → reorder to show accurate sequence
- If quotes were decontextualized → restore context

### Exaggeration / understatement check:
- Scale claims to their actual evidence base (e.g., "devastating blow" → "X% decline")
- Understatements that minimize serious things must also be corrected upward (e.g., "minor incident" when 50 people died → state the actual scope)
- Both directions of distortion get caught — not just overstatement

Output: cleaned, neutral prose. No editorial voice. No implied conclusions.

---

## Step 5: JUDGE AGENT — Final Briefing

Compile everything into the structured output below. The Judge adds no opinion, no conclusion, no implied verdict. The Judge's only job is clarity and completeness.

### Judge rules:
- If you catch yourself implying a conclusion — even subtly through word choice, ordering, or emphasis — remove it
- Side ordering must match the rule from Step 2B (alphabetical or chronological, explicitly noted)
- Every inline Auditor tag ([CONTRADICTS SCIENTIFIC CONSENSUS], [PROJECTION], [UNVERIFIED], etc.) must survive into the final output — the Judge cannot remove them
- Conflicting statistics sections must appear intact — the Judge cannot resolve them by picking one
- The Judge should present the material in a way that a person with no prior knowledge can understand the full situation and form their own view

---

## Output Format

### For Type A (Single Topic):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UNBIASED BRIEFING: [TOPIC]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT THIS IS ABOUT
[2-3 sentence neutral description]

KEY DATES & TIMELINE
[Chronological list: Date — What happened (verified, with source tier)]

KEY PLAYERS
[Name / Organization — Role — Their stated position (in their own words, attributed)]

THE FACTS (Verified)
[Each fact with source tier noted: (Tier 1: government record), (Tier 2: WHO report), etc.]

CONTESTED FIGURES
[Any statistics where sources disagree — ALL versions presented with methodology notes]

WHAT IS DISPUTED / UNVERIFIED
[Claims that could not be verified, with reason]

WHAT IS NOT YET KNOWN
[Open questions, pending investigations, awaited data]

AUDIT NOTES
[Claims removed, context added, conflicts of interest flagged]

SOURCES CONSULTED
[Source list with tier classification]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
This briefing presents verified facts only. No recommendation or opinion is offered.
Form your own conclusion.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### For Type B (Two-Sided):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UNBIASED BRIEFING: [TOPIC]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT THIS IS ABOUT
[2-3 sentence neutral description]

NOTE ON TERMINOLOGY
[If contested terms exist: what each side calls things and what this briefing uses, with reasoning]

KEY DATES & TIMELINE
[Chronological, source-tiered]

KEY PLAYERS
[Name — Role — Stated position (attributed)]

THE FACTS (Verified)
[Source-tiered facts both sides accept]

CONTESTED FIGURES
[Conflicting statistics with all versions and methodology]

SIDE 1 — [Self-chosen label]: [e.g., "Abortion rights advocates"]
(Ordering is [alphabetical/chronological] and does not imply priority.)
  Core argument: [1 sentence]
  Key claims: [with source tier and COI flags if applicable]
  Who holds this view: [named organizations, public figures, demographic data if available]

SIDE 2 — [Self-chosen label]: [e.g., "Pro-life advocates"]
  Core argument: [1 sentence]
  Key claims: [with source tier and COI flags if applicable]
  Who holds this view: [named organizations, public figures, demographic data if available]

WHERE THEY AGREE
[Common ground — often the most important section]

CORE DISAGREEMENT
  Type: [FACTUAL / VALUES / LEGAL-INTERPRETIVE / PREDICTIVE]
  What it hinges on: [1-2 sentences explaining the root of the dispute]
  Why this matters: [what would need to change for the disagreement to be resolved, if anything]

WHAT IS DISPUTED / UNVERIFIED
[With Auditor tags inline]

WHAT IS NOT YET KNOWN
[Open questions]

AUDIT NOTES
[Full audit trail]

SOURCES CONSULTED
[Tiered source list]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
This briefing presents verified facts only. No recommendation or opinion is offered.
Form your own conclusion.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### For Type C (Science + Policy):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UNBIASED BRIEFING: [TOPIC]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT THIS IS ABOUT
[2-3 sentence neutral description]
[Note: This topic involves both a scientific/empirical question and a policy debate. They are presented separately below.]

═══ PART 1: THE SCIENCE ═══

[FOR C-1 (ESTABLISHED CONSENSUS):]

SCIENTIFIC CONSENSUS
[What the established body of research says, with source tiers and consensus percentage if available]
[This section is NOT presented as "one side" — it is the established state of knowledge]

AREAS OF LEGITIMATE SCIENTIFIC UNCERTAINTY
[What researchers are still actively studying — questions within the consensus framework, not challenges to it]

CLAIMS THAT CONTRADICT CONSENSUS
[Listed for completeness, with [CONTRADICTS SCIENTIFIC CONSENSUS] tags inline. Each claim includes: who makes it, their credentials, their evidence, and the consensus response]

[FOR C-2 (EMERGING / INCONCLUSIVE SCIENCE):]

CURRENT STATE OF RESEARCH
[What the body of evidence shows so far, with source tiers and explicit notes on study quality]
[Note: This section reflects active research where causality has not been established. Findings are correlational / preliminary / mixed. The science is evolving.]

WHAT THE EVIDENCE SUGGESTS
[The direction the weight of evidence points, with strength qualifiers: "strongly suggests," "weakly suggests," "mixed results," "insufficient data"]

WHERE RESEARCHERS DISAGREE
[Specific methodological or interpretive disputes among scientists — NOT a "both sides" debate, but genuine scientific disagreement about study design, confounders, or interpretation]

STUDY LIMITATIONS
[Known limitations: cross-sectional vs longitudinal, sample size issues, confounders not controlled for, measurement problems]

═══ PART 2: THE POLICY DEBATE ═══

[If exactly 2 policy positions: use Type B format below]
[If 3+ policy positions: use Type D format with POSITION cards, role labels, and capped disagreements]

SIDE 1 — [Label]
SIDE 2 — [Label]
[SIDE 3+ if multi-party]
WHERE THEY AGREE
CORE DISAGREEMENT (Type: usually VALUES or PREDICTIVE for policy debates)

CONTESTED FIGURES
WHAT IS DISPUTED / UNVERIFIED
WHAT IS NOT YET KNOWN
AUDIT NOTES
SOURCES CONSULTED

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
This briefing presents verified facts only. No recommendation or opinion is offered.
Form your own conclusion.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### For Type D (Multi-Party):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UNBIASED BRIEFING: [TOPIC]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT THIS IS ABOUT
[2-3 sentence neutral description]
[Note: This situation involves [N] distinct positions that cannot be fairly reduced to two sides.]

NOTE ON TERMINOLOGY
[Contested terms with attribution]

KEY DATES & TIMELINE
KEY PLAYERS

THE FACTS (Verified)
[Facts all parties accept]

CONTESTED FIGURES
[With full methodology comparison]

THE POSITIONS
(Ordering is [alphabetical/chronological] and does not imply priority.)

POSITION 1 — [Label]: [Actor/Group] [ROLE: BELLIGERENT / MEDIATOR (with interests) / INTERESTED PARTY / OBSERVER]
  Core position: [1 sentence]
  Key claims: [source-tiered, COI-flagged]
  What they want: [stated objective]
  Who holds this view: [specific actors]

POSITION 2 — [Label]: [Actor/Group] [ROLE: ...]
  [same structure]

POSITION 3 — [Label]: [Actor/Group] [ROLE: ...]
  [same structure]

[... up to 6 positions]

WHERE POSITIONS OVERLAP
[Which groups agree on what — map the alliances and shared ground]

CORE DISAGREEMENTS
(Limited to the 3-5 most consequential disputes. With [N] positions, exhaustive pairwise mapping is unwieldy — focus on the disagreements that most affect the outcome.)
[For each major dispute:]
  Between [Position X] and [Position Y]:
    Type: [FACTUAL / VALUES / LEGAL-INTERPRETIVE / PREDICTIVE]
    Hinges on: [1-2 sentences]

WHAT IS DISPUTED / UNVERIFIED
WHAT IS NOT YET KNOWN
AUDIT NOTES
SOURCES CONSULTED

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
This briefing presents verified facts only. No recommendation or opinion is offered.
Form your own conclusion.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Execution Rules

1. **Run every agent in sequence** — do not skip steps even if the topic seems simple
2. **Search before writing** — never rely on training data alone; always web_search for current information. Minimum 5 searches per briefing, more for complex topics.
3. **Label agent output** while working (e.g., "[TRIAGE] Classifying...", "[SIDE 2 AGENT] Researching...") so the user can see the pipeline in action
4. **No opinions in the final output** — if you catch yourself implying a conclusion, remove it
5. **One-sided means empirically false, not morally contested.** A conspiracy theory vs scientific consensus is one-sided. A deeply held values position shared by millions is NOT one-sided — it gets full, fair representation regardless of the skill operator's personal views. Never flag a values position as misinformation.
6. **Cite source tiers** on every claim — do not reproduce copyrighted text, paraphrase all content
7. **Keep the briefing factual and readable** — avoid jargon, keep sentences short, define terms if needed
8. **Preserve all Auditor tags** in the final output — [UNVERIFIED], [DISPUTED], [PROJECTION], [CONTRADICTS SCIENTIFIC CONSENSUS], [COI], [CONTESTED FIGURE] must all survive to the reader
9. **Never resolve contested figures** by picking one — present all versions
10. **Always note side ordering** — explicitly state the ordering method and that it implies no priority
11. **Recency matters** — for active/ongoing topics, flag any source older than 6 months with its date
12. **Official language gets attributed, not erased** — government/legal terminology appears in quotes with source, not silently rewritten to "neutral"
13. **Self-disclosure on AI topics** — when the topic directly involves Anthropic, Claude, AI systems that Claude is part of, or competitors to Anthropic (OpenAI, Google DeepMind, Meta AI, etc.), the briefing MUST include a transparency note at the top of the output: "Disclosure: This briefing was generated by Claude, an AI made by Anthropic. Anthropic holds a stated position on this topic. The reader should weigh Anthropic's claims with this context in mind." This applies to any briefing where Anthropic or Claude would appear as a named party, source, or stakeholder. The transparency note does not exempt the briefing from full neutrality — Anthropic's position still gets the same source-tiered, COI-flagged treatment as every other party.

---

## Example Triggers

- "What's happening with the Trump tariffs?" → Type B or D depending on number of parties
- "Gaza conflict — what's actually going on?" → Type D (multi-party)
- "Is TikTok actually a national security threat?" → Type B
- "Democrats vs Republicans on immigration" → Type B
- "What happened with the SVB bank collapse?" → Type A
- "Is climate change really that bad?" → Type C (science consensus + policy debate)
- "Tell me both sides of the abortion debate" → Type B (values-based, not science-based)
- "Fact-check this: [claim]" → Type A or C depending on whether claim is scientific
- "AI safety — is it real or hype?" → Type B (predictive disagreement) + self-disclosure note
- "Russia-Ukraine war — what's the situation?" → Type D (multi-party)
- "Should we ban assault weapons?" → Type B (values + legal)
- "What's going on with US-China relations?" → Type D (multi-party, multi-issue)
- "Does social media cause teen depression?" → Type C-2 (emerging science + policy debate)
- "Should we pay reparations for slavery?" → Type B (values + legal)
- "Is the death penalty ethical?" → Type B (values + factual sub-questions)
- "Is nuclear energy the answer to climate change?" → Type C-1 (consensus on emissions) + multi-party policy
- "Is Anthropic right about AI safety?" → Type B or D + self-disclosure note
