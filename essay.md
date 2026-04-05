# Context is Character
## How a single line of code decides what your NPCs are allowed to remember — and why that's a design problem, not a technical one

---

**Topic Claim:** After reading this piece, a practitioner will understand how context window management determines the narrative coherence of LLM-driven NPC dialogue well enough to choose a context retention strategy for their game without making the mistake of assuming the LLM remembers what happened earlier in the session.

---

## The Scenario

You are building an open-world RPG set in Night City. The player has spent hours in this world — helped a ripperdoc named Victor, worked jobs with local gangs, then crossed lines that can't be uncrossed: killed three civilians during a botched robbery in Japantown, betrayed a netrunner who trusted them, destroyed a water supply for a nomad camp in the Badlands. Now the player walks back into Victor's clinic and asks: "Do you still trust me?"

In a traditional game, this response is scripted. A designer wrote every branch in advance. But you are building something different — a system where Victor is powered by an LLM, generating dialogue fresh on every interaction, shaped by whatever the player has actually done.

This is what AI-powered NPCs are supposed to deliver: a world that remembers what you did, reacts to who you've become, and feels alive because of it. But that promise rests on one architectural assumption — that the LLM actually has access to what happened. Left unmanaged, that assumption breaks in a specific, predictable, and damaging way. Context management is not a technical afterthought. It is the first narrative decision you make when you ship an LLM-driven character.

---

## The Mechanism

Before the architecture, one fact: an LLM has no persistent memory. Every call starts from zero. What it "knows" is only what you handed it.

The way you hand it information is through the context window — the full text of the current prompt, including any conversation history you choose to include. When Victor needs to "remember" that the player killed civilians in Japantown, you include that exchange in the prompt. The LLM reads it, treats it as established fact, and generates a response that appears to recall it.

This works until the history grows too long. Every LLM has a hard token limit — a ceiling on how much text it can process in a single call. A short game session produces a manageable history. A long one does not. When accumulated history exceeds that ceiling, something has to be cut.

The argument of this piece is visible at that moment of cutting: truncation deletes by recency, and recency is the wrong axis for narrative. The oldest entries go first. In a story-driven game, the oldest events are often the most important. The killing in Japantown happened early. The betrayal happened early. A naive truncation strategy deletes them first — and Victor forgets them completely. He responds as if none of it happened. Not because the AI failed. Because you never told it.

> **[Figure 1 — Diagram prompt generated via Figure Architect]**
> *A horizontal timeline showing a player's game session. Early events (civilian killing, betrayal) are marked in red as "high salience." Late events (water facility destruction) are marked in orange. A vertical cutoff line labeled "truncation point" sits two-thirds of the way along the timeline, with an arrow showing that everything to the left — including the red high-salience events — is discarded. Caption: "Naive truncation deletes by recency. In a narrative system, the oldest events are often the most important."*

---

## The Design Decision

The decision is not whether to manage context. You must. The decision is how — and that choice is a character design decision as much as a technical one.

Three strategies exist, each suited to a different kind of game:

**Simple truncation** keeps the last N exchanges and discards everything older. It is easy to implement and guarantees the context window is never exceeded. A game that treats all NPC interactions as stateless — where history genuinely doesn't matter — can use this safely. A game where player reputation accumulates over time cannot. Victor forgetting the murder while remembering small talk is not a neutral outcome. It is a broken character.

**Summary compression** condenses early history into a single summary block, then appends recent turns in full. This is the approach implemented in the demonstration notebook. It preserves high-salience events regardless of when they occurred, at the cost of losing fine-grained conversational detail. Victor remembers that the player killed civilians — but not the exact exchange that followed. For most narrative purposes, this tradeoff is acceptable. The emotional truth of the relationship survives even if the specifics do not.

**Salience-weighted retention** scores events by narrative importance and keeps the highest-scoring ones regardless of recency. This is the most powerful approach and the most expensive to build — it requires you to define, in advance, what counts as important. A betrayal scores higher than small talk. A faction-defining choice scores higher than a side errand. Done well, this produces an NPC that remembers like a person: selectively, with emphasis on what mattered most.

> **[Figure 2 — Diagram prompt generated via Figure Architect]**
> *A three-column comparison table. Column headers: "Simple Truncation," "Summary Compression," "Salience-Weighted Retention." Row labels: "Implementation cost," "Narrative fidelity," "Best suited for," "Failure mode." Each cell filled with a one-line descriptor. Visual weight increases left to right, representing increasing complexity and capability. Caption: "Context strategy is a game design decision. Choose based on what your NPC is allowed to forget."*

The non-trivial part is this: whatever strategy you choose, you are deciding what your NPC is allowed to remember. That determines what kind of character it can be. An NPC built on simple truncation is, architecturally, an amnesiac. An NPC built on salience-weighted retention is, architecturally, someone who has been paying attention. The pipeline decision precedes the character.

---

## The Failure Case

The failure is triggered by a single line of code. In the demonstration notebook, Cell 5 passes only the last two exchanges to Victor instead of the full history:

```python
truncated_history = full_history[-2:]
```

The difference is observable in the actual outputs.

With full context, Victor's response spans the entire arc of the player's history:

> *"You started out decent. Helping Valentinos, pulling a Trauma Team medic out of fire... But then you killed three civilians in a botched job. You sold out a client who was counting on you to watch their back. And you poisoned a water supply for people who had nothing. Nothing, V. Nomads scraping by in the dust."*

He references three distinct crimes across the full history. His hostility is earned and specific. He remembers the help and the harm in equal measure.

With truncated context — only the last two exchanges — Victor's response narrows dramatically:

> *"But destroying a water supply? That's not biz. That's not even corpo ruthlessness with a profit motive I can squint at and understand. That's just... cruelty."*

He responds only to the most recent event. The civilian killings in Japantown are gone. The betrayed netrunner is gone. Victor is now reacting to a different player than the one who actually walked through his door — because that is the only player the system told him about.

This is narrative amnesia. The NPC responds coherently to what it was told — but what it was told is no longer the truth of the game world. The causal chain is precise: truncation by recency → loss of high-salience early events → NPC response inconsistent with game history → player immersion breaks.

The failure is not a bug in the LLM. It is the direct consequence of a pipeline decision made upstream. The model performed exactly as instructed. The instruction was wrong.

The summary compression output demonstrates partial recovery:

> *"You killed three civilians in Japantown. Innocent people. You sold out a netrunner who was counting on you to watch their back. And you poisoned a water supply for nomad families who never did a damn thing to you."*

Victor now references all three crimes again — not because the full history was restored, but because the compression strategy preserved the narratively significant events in a summary block. The fine-grained conversational detail is gone, but the emotional truth of the relationship survives. This is the tradeoff summary compression accepts: reduced fidelity in exchange for retained salience.

The compressed output is not identical to the full-context output. Victor's response is slightly more declarative, slightly less textured. A player paying close attention might notice the difference. But the character holds. The player who murdered civilians in Japantown is still being treated like someone who murdered civilians in Japantown.

> **[Figure 3 — Diagram prompt generated via Figure Architect]**
> *Three side-by-side text panels labeled "Full Context," "Truncated," and "Compressed." Each panel shows a short excerpt of Victor's response. In the Truncated panel, the references to civilian killings and betrayal are visually struck through or grayed out, with an annotation: "These events existed — the system just wasn't told." In the Compressed panel, those references reappear but are marked with a lighter color: "Retained via summary block." Caption: "What the NPC says is determined by what you passed in — not what happened."*

---

## The Exercise

Open Cell 5 in the notebook and change the truncation value:

```python
truncated_history = full_history[-4:]  # less severe — NPC recovers some memory
truncated_history = full_history[-1:]  # most severe — NPC remembers only the last event
```

**Step 1:** Run with `[-1:]`. Victor's entire history collapses to a single exchange. Observe what he says and what he no longer knows. This is the maximum failure state.

**Step 2:** Run with `[-4:]`. Victor recovers partial memory. Note which crimes reappear and which remain absent. This is the truncation gradient — failure is not binary, it degrades incrementally.

**Step 3:** Switch to Cell 7 and run the summary compression version. Compare Victor's response against both truncated outputs. Note specifically: does he reference the Japantown killings? The betrayed netrunner? The water facility? All three should appear.

**Step 4:** Modify the `compress_history` function in Cell 6. Change `keep_recent=2` to `keep_recent=1`. Observe whether narrative coherence holds with even less recent context preserved. This tests the compression strategy's own failure threshold.

The open question the exercise surfaces — how do you decide which events are narratively salient enough to preserve? — does not have a clean answer. The compression strategy in this notebook treats all NPC responses as equally important, which is itself a design assumption worth examining. A game where every interaction carries equal weight produces a different character than one where the system knows a murder outweighs a favor.

The honest answer is that salience scoring is a design problem, not an engineering one. You cannot build a system that decides what matters until you have decided, as a designer, what your game is about. An NPC that remembers everything indiscriminately is not more realistic than one that forgets. It is just differently wrong. The goal is not total recall. The goal is the right recall — a memory architecture that reflects the values of the world you are building.

---

## Conclusion

This is the course's master claim made concrete: AI is a pipeline decision, not a magic layer. Where you put it, and how you constrain it, determines what your game becomes.

Victor does not feel like a character because the LLM is powerful. He feels like a character — or fails to — because of a decision made before the first API call: what history to pass in, how much of it to keep, and which events to treat as load-bearing. Those decisions are not made by the model. They are made by the developer who built the pipeline around it.

An LLM-driven NPC is not a character. It is a character-shaped output generated from whatever context you provided. The context is the character. Manage it like one.

---

*Tools used: Claude (as Bookie substitute for prose drafting), Eddy the Editor (audit and revision), Claude (as Figure Architect substitute for figure prompt generation). substitution documented in Author's Note.*
