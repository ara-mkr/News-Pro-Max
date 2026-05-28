# Examples and Trigger Phrases

## How to Use

Ask Claude any question about current events, news, politics, science, or controversy. The skill activates automatically on relevant queries.

## Example Queries by Type

### Type A: Single Topic (factual events, no active debate)

```
"What happened with the collapse of Silicon Valley Bank?"
"What is the current legal status of Julian Assange?"
"What happened during the 2021 Myanmar military coup?"
"What is the factual timeline of the Nord Stream pipeline sabotage?"
"What happened with the FTX collapse?"
"What is the current status of Taiwan's diplomatic recognition?"
"What happened during the Wagner Group mutiny?"
"What is the factual record of the Titan submersible disaster?"
```

### Type B: Two-Sided (two clearly opposing positions)

```
"Should the United States ban TikTok?"
"Should reparations be paid to descendants of American slaves?"
"Is the death penalty effective and just?"
"Should the US adopt a universal basic income?"
"Is Edward Snowden a traitor or a whistleblower?"
"Should the US maintain military support for Ukraine?"
"Is affirmative action in university admissions ethical?"
```

### Type C-1: Established Science + Policy Debate

```
"Is climate change real and what should we do about it?"
"Should nuclear energy be a central part of the climate solution?"
"Do vaccines work and should they be mandated?"
```

### Type C-2: Emerging/Inconclusive Science + Policy Debate

```
"Does social media cause teen mental health problems?"
"Is mass immigration net positive or negative economically?"
"Should psychedelic drugs be legalized for therapeutic use?"
"Do microplastics cause health problems?"
"Does remote work reduce productivity?"
```

### Type D: Multi-Party (3+ distinct positions)

```
"What's actually going on with the Gaza conflict?"
"Should AGI development be paused or regulated?"
"Should the EU break up or devolve power?"
"Should wealthy nations pay climate reparations?"
"Is Zionism compatible with Palestinian self-determination?"
"Should CRISPR gene editing in human embryos be permitted?"
"Should nuclear-weapon states be required to disarm?"
"What's going on with US-China relations?"
"Russia-Ukraine war -- what's the situation?"
```

## What the Output Looks Like

Every briefing follows a structured format. Here are the key sections:

### For all types:
- **WHAT THIS IS ABOUT**: 2-3 sentence neutral description
- **NOTE ON TERMINOLOGY**: When terms are contested (appears when needed)
- **KEY DATES AND TIMELINE**: Chronological, with source tiers on each entry
- **KEY PLAYERS**: Name, role, stated position (in their own words)
- **THE FACTS (Verified)**: Source-tiered, accepted by all parties
- **CONTESTED FIGURES**: When statistics disagree across sources -- ALL versions shown
- **WHAT IS DISPUTED / UNVERIFIED**: With Auditor tags inline
- **WHAT IS NOT YET KNOWN**: Open questions
- **AUDIT NOTES**: Full audit trail
- **SOURCES CONSULTED**: Tiered list

### Additional sections for adversarial topics (B/C/D):
- **SIDE 1 / SIDE 2 / POSITION 1-N**: Each presented in its best light
- **WHERE THEY AGREE**: Common ground
- **CORE DISAGREEMENT**: Typed as FACTUAL, VALUES, LEGAL-INTERPRETIVE, or PREDICTIVE

### Additional for Type C:
- **PART 1: THE SCIENCE** (consensus or emerging research)
- **PART 2: THE POLICY DEBATE** (adversarial treatment)

### Additional for Type D:
- Role labels on each position: BELLIGERENT, MEDIATOR (with interests), INTERESTED PARTY, OBSERVER

## Inline Tags That Appear in Output

These Auditor tags survive to the final briefing and cannot be removed by the Judge:

| Tag | Meaning |
|-----|---------|
| `[UNVERIFIED -- single source]` | Claim based on one source with no corroboration |
| `[UNVERIFIED -- single-tier sourcing]` | Claim supported only by Tier 4-5 sources |
| `[DISPUTED -- contradicts: source]` | Claim contradicts established facts |
| `[UNCONFIRMED]` | Rumor, speculation, or unconfirmed report |
| `[CONTEXT ADDED]` | Auditor added missing context |
| `[REMOVED -- reason]` | Demonstrably false claim removed |
| `[PROJECTION]` | Future prediction, not a verified fact |
| `[COI: description]` | Source has a conflict of interest |
| `[CONTESTED FIGURE]` | Multiple sources report different numbers |
| `[CONTRADICTS SCIENTIFIC CONSENSUS]` | Claim contradicts peer-reviewed consensus |

## Source Credibility Tiers

| Tier | Description | Examples |
|------|-------------|---------|
| 1 | Primary sources | Government records, court filings, legislation, peer-reviewed studies, raw data |
| 2 | Institutional | Major research institutions, WHO, CDC, NASA, IPCC, UN bodies |
| 3 | Quality journalism | Wire services (Reuters, AP, AFP), major outlets with editorial standards |
| 4 | Expert commentary | Named experts with relevant credentials, academic op-eds |
| 5 | Advocacy/interest groups | Think tanks, NGOs, industry groups, partisan media |
