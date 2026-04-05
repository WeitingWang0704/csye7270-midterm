# Context is Character
### CSYE 7270 — Take-Home Midterm
**Category E: Emerging or Novel AI Uses in Game Dev**

---

## Topic Claim

After reading this piece, a practitioner will understand how context window management determines the narrative coherence of LLM-driven NPC dialogue well enough to choose a context retention strategy for their game without making the mistake of assuming the LLM remembers what happened earlier in the session.

---

## What This Is

A proof-of-concept demonstration showing how context window management affects narrative coherence in LLM-driven NPC dialogue, using the Anthropic API to simulate Victor Vector, a ripperdoc NPC from Cyberpunk 2077.

Three conditions are tested and compared:
- **Full context** — NPC has access to complete game history
- **Truncated context** — naive truncation causes narrative amnesia
- **Summary compression** — corrected approach restores coherence

---

## Repository Structure

```
csye7270-midterm/
├── npc_narrative_context.ipynb   # Demo notebook
├── essay.md                       # Technical essay
└── README.md                      # This file
```

---

## How to Run

1. Clone this repo
2. Install dependencies:
   ```bash
   pip install anthropic jupyter
   ```
3. Open the notebook:
   ```bash
   jupyter notebook npc_narrative_context.ipynb
   ```
4. Replace `YOUR_API_KEY_HERE` in Cell 1 with your Anthropic API key
5. Run all cells: **Run → Run All Cells**

---

## Triggering the Failure Case

In Cell 5, change the truncation value:

```python
truncated_history = full_history[-2:]  # default — triggers failure
truncated_history = full_history[-1:]  # most severe failure
truncated_history = full_history[-4:]  # less severe
```

Observe how Victor's response degrades as more history is removed.

---

## Human Decision Node

Located in **Cell 6**. The AI proposed simple truncation as the context management strategy. I rejected it because truncation discards by recency, not by narrative importance — the most significant events occur earliest in the history and would be deleted first. My decision was summary compression, which preserves high-salience events regardless of when they occurred.

---

## Tools Used

- **Bookie the Bookmaker** — used to draft prose bridging intuition to mechanism; substituted with Claude
- **Eddy the Editor** — used to audit draft against rigorous standards; flagged structural issues in Design Decision section and ending
- **Figure Architect** — used to generate figure prompts for high-assertion claims; substituted with Claude
- **Anthropic API** — claude-opus-4-6 for NPC dialogue generation in demo
