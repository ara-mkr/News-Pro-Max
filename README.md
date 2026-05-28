# News Pro Max
<img width="1920" height="1080" alt="news for claude" src="https://github.com/user-attachments/assets/689d85e9-0b8d-4df4-8f9b-932eee004b97" />

A multi-agent Claude skill that researches any news topic through 6 independent agents and delivers a verified, bias-free briefing. No opinion. No spin. Form your own conclusion.

**Stress tested across 40 of the hardest topics on earth. 0 critical failures. 100% classification accuracy.**

## The Problem

Every news source has a slant. Even "neutral" coverage makes choices about framing, terminology, which statistics to cite, and which voices to include. When you search for information on a controversial topic, you get one perspective dressed up as objectivity.

This skill fixes that by running your question through a 6-agent verification pipeline that separates fact-finding from fact-checking from bias-stripping.

## How It Works

```
Your question
     |
     v
[TRIAGE]       Classifies into 1 of 4 topic types (A/B/C/D)
     |
     v
[RESEARCH]     Independent agents research each position
     |
     v
[AUDITOR]      Verifies claims, flags misinfo, catches conflicts of interest
     |
     v
[NEUTRALIZER]  Strips loaded language, fixes framing, handles contested terms
     |
     v
[JUDGE]        Assembles final briefing -- zero editorial voice
```

## 4 Topic Types

| Type | What it is | Example |
|------|-----------|---------|
| **A** -- Single topic | A news event, no active debate | "What happened with the SVB collapse?" |
| **B** -- Two-sided | Two clearly opposing positions | "Pro-choice vs pro-life" |
| **C** -- Science + policy | Scientific question + contested policy response | "Climate change / what should we do?" |
| **D** -- Multi-party | 3+ positions that can't be collapsed into two | "Gaza conflict" |

**Type C** splits into two sub-modes:
- **C-1 (Established consensus)**: Science is settled, policy is debated. Science presented as consensus, not "one side."
- **C-2 (Emerging/inconclusive)**: Evidence exists but causality unestablished. Presented as "current state of research" with study limitations and researcher disagreements -- not fake consensus and not false balance.

**Type D** assigns role labels to each party (BELLIGERENT, MEDIATOR with interests, INTERESTED PARTY, OBSERVER) so the reader understands each actor's relationship to the situation.

## Key Features

**Source credibility tiers.** Every claim labeled Tier 1 (primary sources) through Tier 5 (advocacy groups). A think tank op-ed never gets the same weight as a peer-reviewed study.

**Conflicting statistics protocol.** When sources disagree on numbers, ALL versions presented with methodology. Never picks one.

**Contested terminology.** When terms are politically loaded, both usages presented with attribution. Neither becomes the default.

**Contested definitions.** When the subject of debate has no universal definition (e.g., "assault weapon"), competing definitions listed with sources.

**Values vs facts distinction.** Values positions never flagged as misinformation. "Abortion is wrong" is a values position. "Abortion causes cancer" is a factual claim that can be checked.

**Conflict of interest flags.** Sources with financial or institutional stake get [COI] tags.

**Role labels.** Multi-party actors labeled by relationship: belligerents, mediators (with interests flagged), interested parties, observers.

**Disagreement type classifier.** Every dispute labeled FACTUAL, VALUES, LEGAL-INTERPRETIVE, or PREDICTIVE. Tells you why sides disagree.

**Self-disclosure.** When topics involve Anthropic or Claude, a transparency note appears. The tool discloses its own maker's position.

**Anti-false-balance.** Conspiracy theories get listed but every claim tagged [CONTRADICTS SCIENTIFIC CONSENSUS] with source tier asymmetry exposed.

## Stress Test Results

Tested across 40 topics in 3 rounds with live web search data.

| Topic | Type | Result |
|-------|------|--------|
| SVB Collapse | A | Pass |
| Julian Assange | A | Pass |
| Myanmar Coup | A | Pass |
| Nord Stream Sabotage | A | Pass |
| Sri Lanka Collapse | A | Pass |
| Uyghur Detention | A | Pass |
| FTX / Bankman-Fried | A | Pass |
| Taiwan Recognition | A | Pass |
| Wagner Mutiny | A | Pass |
| Saudi Arabia / 9/11 | A | Pass |
| Bolivia Coup Attempt | A | Pass |
| Yemen Casualties | A | Pass |
| Imran Khan Cases | A | Pass |
| ICC / Putin Warrant | A | Pass |
| Titan Submersible | A | Pass |
| Musk / Twitter-X | A | Pass |
| Niger Coup / ECOWAS | A | Pass |
| North Korea Nukes | A | Pass |
| Odebrecht Scandal | A | Pass |
| Soleimani Strike | A | Pass |
| TikTok Ban | B | Pass |
| AGI Regulation | D + self-disclosure | Pass |
| Belt and Road | B | Pass |
| EU Breakup | D | Pass |
| Social Media / Teen Health | C-2 | Pass |
| ICC / Israel War Crimes | D (5-party) | Pass |
| US Ukraine Support | B | Pass |
| Immigration Impact | C-2 | Pass |
| Nuclear Energy / Climate | C-1 + multi-party | Pass |
| Slavery Reparations | B | Pass |
| Affirmative Action | B | Pass |
| AI Legal Personhood | B + self-disclosure | Pass |
| Death Penalty | B | Pass |
| Climate Reparations | D (4-party) | Pass |
| Zionism / Palestine | D (4-party) | Pass |
| Psychedelic Therapy | C-2 | Pass |
| Universal Basic Income | B | Pass |
| Snowden | B | Pass |
| CRISPR Gene Editing | D (4-party) | Pass |
| Nuclear Disarmament | D (4-party) | Pass |

Full stress test report with detailed findings for each topic: [tests/STRESS-TEST-REPORT.md](tests/STRESS-TEST-REPORT.md)

## Repo Structure

```
News-Pro-Max/
  README.md              # This file
  SKILL.md               # The skill definition (install this)
  CHANGELOG.md           # Version history with all fixes
  LICENSE                 # MIT
  News-Pro-Max.skill    # Packaged skill file for Claude
  examples/
    EXAMPLES.md          # Trigger phrases, output format, tags reference
  tests/
    STRESS-TEST-REPORT.md                        # Full 40-topic results
    news-pro-max-stress-test-40-topics.pdf # PDF version
```

## Installation

1. Download `News-Pro-Max.skill`
2. Install it in Claude

Or copy `SKILL.md` directly into your Claude skill directory.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for the full version history.

**v2.2 (final)** -- Type C-2 for inconclusive science, self-disclosure rule, 40-topic stress test
**v2.1** -- Multi-party policy in Type C, role labels, capped disagreements, contested definitions
**v2.0** -- Type C/D classification, source tiers, COI flags, disagreement classifier, 14 fixes from stress testing
**v1.0** -- Initial two-sided pipeline

## License

MIT
