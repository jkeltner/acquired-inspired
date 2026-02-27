# Backbone: From Breakthrough to Built-In

An AI-powered production pipeline for a deeply researched, narrative-driven podcast about how technologies actually spread — not the invention myth, but the messy middle.

**Hosts:** Jeff Keltner and Cyrus Mistry

---

## The Show

Most tech content skips from "invention" to "winner." Backbone tells the **diffusion story** — the messy middle that gets glossed over: who resisted and why, what complementary infrastructure had to exist first, the tipping point from novelty to inevitability, and the second-order effects that nobody predicted.

Each episode runs 90–120 minutes and follows a consistent structure:

| Section | Length | What It Does |
|---------|--------|--------------|
| **The Hook** | 12–18 min | Cold open scene, "By the Numbers" stats, the world before this technology, wave preview |
| **The Waves** (2–4) | 15–25 min each | The diffusion story, chapter by chapter — breakthrough, resistance, what changed |
| **Built In** | 15–20 min | Full arc, the Backbone Test, what the story teaches |

**The Backbone Test** closes every episode — five questions applied to every technology:
1. *Invisible?* — Has it become infrastructure you only notice when it fails?
2. *What depends on it?* — Map the dependency chain
3. *What's the hidden cost?* — Energy, labor, environment, inequality, fragility
4. *Could we go back?* — How deep is the dependency?
5. *What's next?* — What's still evolving behind the scenes?

---

## How the Pipeline Works

This repo is an AI agent pipeline. Each file in `roles/` is a complete briefing for a Claude Code agent. Agents read `CLAUDE.md` (shared context) plus their role file, then produce specific deliverables that feed the next stage.

```
Topic Selection (human)
        │
        ▼
Research Director ──── Phase 1: Broad overview research
        │
        ▼
Narrative Architect ── Story blueprint (waves, anchor stories, structure)
        │
        ▼
Research Director ──── Phase 2: Chapter-by-chapter deep dives
        │
        ▼
Script Writer ───────── Full TTS-ready dialogue, chapter by chapter
        │
        ▼
Editor ──────────────── Pacing, continuity, voice consistency
        │
        ▼
Fact Checker ────────── Verify claims, stats, quotes
        │
        ▼
Producer ────────────── Final assembly, metadata, companion content
        │
        ▼
[Jeff + Cyrus record feedback conversation → feedback.txt]
        │
        ├─────────────────────────┐
        ▼                         ▼
Profile Updater            Pipeline Reviewer
(async — proposes           (live session — edits
host profile edits)          applied in real time)
```

---

## The Roles

### Research Director
Runs **twice** per episode. Phase 1 casts a wide net — source landscape, key figures, anchor story candidates, resistance, enabling conditions. Phase 2 goes deep on the specific chapters the Narrative Architect has defined. The quality of everything downstream depends on what the Research Director surfaces.

### Narrative Architect
Turns raw research into a **blueprint** — the binding creative contract. Defines wave boundaries, selects and places anchor stories, assigns hosts by worldview fit, specifies the "road not taken" for each wave, and identifies what the episode's diffusion story teaches. Every downstream agent builds from the blueprint.

### Script Writer
Writes fully scripted, TTS-ready dialogue for ElevenLabs v3. Works chapter by chapter, following the blueprint. Manages host knowledge division (one host drives each wave, the other discovers), primes every anchor story with host reaction, and writes How It Works sections as dialogue rather than monologue.

### Editor
Reviews the assembled script as a continuous episode. Catches continuity gaps, pacing problems, voice drift, planted callbacks that never land, and transitions that complete rather than propel. Also runs a systematic voice consistency check against the host profiles.

### Fact Checker
Verifies claims, statistics, and quotes against sources. Produces a report with confidence levels and flags anything the hosts should double-check before recording.

### Producer
Assembles the final episode: the complete script, episode metadata, show notes, and social content. The deliverable package for production.

### Profile Updater *(post-episode)*
Reads `feedback.txt` from the post-episode conversation and proposes specific, evidence-backed edits to the host profiles. Outputs a proposals file for review — host profile changes are sensitive enough to warrant both Jeff and Cyrus reviewing before anything is applied.

### Pipeline Reviewer *(post-episode, live session)*
A live Claude Code session rather than an autonomous agent. Jeff opens a session with `feedback.txt`, works through findings one at a time — each finding presented as a specific proposed edit, applied immediately on approval. The output is committed changes to the repo, not a proposals document.

---

## The Host Profiles

`hosts/jeff.md` and `hosts/cyrus.md` are detailed personality profiles — background, communication style, verbal patterns, areas of expertise, and how they interact. The Script Writer reads these before writing a single line of dialogue. They evolve over time through the Profile Updater feedback loop.

**Jeff** brings a technology industry and policy background. His instincts run toward institutional reform — he tends to see resistance as solvable through better leadership or policy, and he builds arguments incrementally, hedging with "I think" and grounding abstractions in examples.

**Cyrus** brings a structural, systems-level lens. His instincts run toward disruption as a structural force — he tends to see resistance as symptomatic rather than fixable, and he speaks with density and directness, using "like" naturally and pivoting mid-thought with "by the way."

Their differing worldviews generate the show's best moments — particularly on the Backbone Test question about hidden costs, and in the "What the Story Teaches" closing beat.

---

## Directory Structure

```
backbone/
├── CLAUDE.md                        ← shared context read by all agents
├── README.md                        ← this file
├── hosts/
│   ├── jeff.md                      ← Jeff's personality profile
│   └── cyrus.md                     ← Cyrus's personality profile
├── roles/                           ← one prompt file per agent
│   ├── research-director.md
│   ├── narrative-architect.md
│   ├── script-writer.md
│   ├── editor.md
│   ├── fact-checker.md
│   ├── producer.md
│   ├── profile-updater.md
│   └── pipeline-reviewer.md
├── templates/                       ← output format contracts
│   ├── blueprint.md
│   ├── research-overview.md
│   └── research-chapter.md
├── episodes/
│   └── {topic}/
│       ├── research/
│       │   ├── overview.md          ← Phase 1 research
│       │   └── chapter-NN-*.md     ← Phase 2 chapter research
│       ├── blueprint.md             ← story structure (binding contract)
│       ├── script/
│       │   ├── chapter-NN-*.txt    ← chapter scripts
│       │   ├── editor-notes.md
│       │   ├── fact-check-report.md
│       │   └── assembled.txt        ← full episode script
│       ├── feedback.txt             ← post-episode Jeff + Cyrus conversation
│       ├── profile-update-proposals.md
│       └── final/                   ← assembled deliverables
└── acquired_transcripts/            ← reference: 10 Acquired podcast transcripts
```

---

## Running an Episode

Each agent is invoked as a Claude Code Task. The typical invocation:

> *"You are the Research Director for Backbone. Read `CLAUDE.md` and `roles/research-director.md`, then run Phase 1 research on [topic]. Save your output to `episodes/[topic]/research/overview.md`."*

Agents are designed to run sequentially, each reading the prior agent's output. The Research Director's Phase 2 and the Script Writer are the most time-intensive runs.

### After the episode is produced
1. Jeff and Cyrus record a free-form feedback conversation and save it as `episodes/{topic}/feedback.txt`
2. Run the **Profile Updater** as an agent to generate profile proposals
3. Open a **Pipeline Reviewer** live session to work through process improvements interactively

---

## Episodes

| Episode | Status | Notes |
|---------|--------|-------|
| Refrigeration | ✅ Pilot complete | Full pipeline run; pilot cut recorded; host profiles need refinement from live recording |

---

## Key Design Principles

**The blueprint is the contract.** The Narrative Architect's blueprint is what every downstream agent builds from. A weak blueprint produces a weak episode regardless of how well the other agents execute.

**Resistance deserves equal time.** The podcast's differentiator is showing why people fought technology adoption and why their arguments were often reasonable. If the resistance section of any wave is shorter than the breakthrough section, the work isn't done.

**Backstory is thesis.** Every key figure needs a formative backstory that causally explains their central decision — not biographical color, but the link between who they were and what they built.

**Contingency over inevitability.** Every wave specifies the Road Not Taken — the competing path, the near-miss, what the world would have looked like if the resistance had won. This is how the diffusion story becomes intellectually honest rather than a winner's narrative.

**The feedback loop compounds.** Each episode's `feedback.txt` improves both the host profiles and the pipeline. Over time, the system gets better at sounding like Jeff and Cyrus and at producing episodes they're proud of.
