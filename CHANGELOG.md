# Changelog

All notable changes to the Unbiased News Skill are documented here.

## v2.2 (Final) - May 2026

### Added
- Type C-2 sub-mode for emerging/inconclusive science (evidence exists but causality unestablished)
- Self-disclosure rule: transparency note when topics involve Anthropic, Claude, or AI competitors
- 5 new trigger examples covering C-2 and self-disclosure cases
- Full C-2 output format with CURRENT STATE OF RESEARCH, WHAT THE EVIDENCE SUGGESTS, WHERE RESEARCHERS DISAGREE, and STUDY LIMITATIONS sections

### Stress Tested
- 40 topics across 3 rounds, 12 categories
- 0 critical failures, 100% classification accuracy

## v2.1 - May 2026

### Added
- Type C policy section now allows multi-party (Type D-style) when 3+ policy positions exist
- Type D positions include role labels: BELLIGERENT, MEDIATOR (with interests), INTERESTED PARTY, OBSERVER
- Type D core disagreements capped at 3-5 most consequential (prevents unwieldy pairwise mapping)
- CONTESTED DEFINITIONS rule in Neutralizer for subjects with no universal definition

### Fixed
- Climate policy debate forced into binary when 4+ positions exist
- Russia-Ukraine mediator roles not distinguished from belligerents
- Assault weapons definition assumed to be universal

## v2.0 - May 2026

### Added
- Type C (science + policy split) triage classification
- Type D (multi-party conflict) triage classification with 3-6 position support
- Source credibility tiers (Tier 1-5) on every claim
- Conflicting statistics protocol: present ALL versions with methodology
- Conflict of interest detection and [COI] tagging
- Disagreement type classifier: FACTUAL, VALUES, LEGAL-INTERPRETIVE, PREDICTIVE
- Speculative/future claim handling: labeled as [PROJECTION], never flagged as misinformation
- Contested terminology protocol for identity labels, movement labels, and event labels
- Official language exception: government/legal terminology preserved with attribution
- Values vs facts distinction: values positions never flagged as misinformation
- Recency weighting: prefer last 6 months for ongoing topics
- Side ordering transparency: ordering method stated, does not imply priority
- Same-metric verification for statistics before comparing across sides
- Official source per side requirement in Researcher

### Fixed
- Gaza conflict forced into binary (was 6-party)
- Climate change false-balanced (science vs denial treated equally)
- Statistics from different methodologies compared as if equivalent
- Loaded language not caught when terms themselves were contested
- Values-based positions incorrectly flagged as misinformation

## v1.0 - May 2026

### Initial Release
- Basic two-sided pipeline: Triage, Researcher/Side Agents, Auditor, Neutralizer, Judge
- Type A (single topic) and Type B (two-sided) classification
- Basic source requirements and neutralization
- Output format with timeline, key players, facts, sides, common ground, disagreement
