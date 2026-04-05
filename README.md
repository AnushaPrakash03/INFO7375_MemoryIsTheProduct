# Chapter 5: Memory Is the Product
### My AskAI's Customer Support Architecture
**Design of Agentic Systems with Case Studies — INFO 7375 Take-Home Midterm**

### YouTube Video Link: https://youtu.be/caaLVtPVicc
---

## What This Is

A production case study (Type B) demonstrating that AI support agent failures root-cause to **memory architecture**, not model capability. The central claim is proven through two real failure cases:

- **Air Canada / Jake Moffatt, February 2024** — BC Civil Resolution Tribunal, $812.02 in damages. Failure mode: absent Tier 5 citation constraint. The agent fabricated a bereavement fare policy from its weights because no retrieval-before-generation requirement existed.
- **Samsung / ChatGPT, April 2023** — Three incidents, proprietary source code and meeting transcripts permanently exited Samsung's organizational boundary. Failure mode: absent Tier 1→Tier 2 write boundary. The write happened automatically, by default.

Both failures are architecturally preventable. This chapter builds the architecture that prevents them.

---

## Repository Structure

```
├── README.md
├── chapter5_v3_EDDY_REVISED.docx       ← Final chapter prose (v3, Eddy-revised)
├── chapter5_authors_note.docx          ← 3-page Author's Note (design choices, tool usage, self-assessment)
├── chapter5_memory_is_the_product_DEMO.ipynb  ← Demo notebook (25 cells, fully runnable)
└── chapter5_memory_is_the_product.pdf  ← PDF copy of the chapter
```

---

## Running the Notebook

### Requirements

```bash
pip install anthropic scikit-learn numpy
```

The notebook runs without an Anthropic API key. The AI Scaffold cell (Cell 5) requires one for the live Claude call; it falls back to a simulated proposal automatically if no key is present. All exercises run without an API key.

### Quick start

```bash
jupyter notebook chapter5_memory_is_the_product_DEMO.ipynb
# Kernel → Run All Cells
```

Full execution takes approximately 10 seconds.

---

## Triggerable Exercises

### Exercise A — Trigger the Amnesiac Pivot (Tier 1 failure)

The Amnesiac Pivot occurs when goal-defining tokens are displaced from the FIFO working memory buffer mid-conversation. The agent closes the session without completing an open refund because the refund context was evicted.

**To trigger the failure:**

In Cell 18, `SupportAgent` is initialized with `max_history_turns=3`. Run the cell. The output will show:

```
🔴 AMNESIAC PIVOT DETECTED
   Refund opened at turn 1
   Refund context EVICTED (max_turns exceeded)
   Session closed WITHOUT completing refund
```

**To observe the fix:**

Cell 19 initializes with `max_history_turns=3, use_goal_pinning=True`. Run the cell. The output will show:

```
✅ Refund completed successfully
   Goal pinning active — goal persisted despite N evictions
```

**The architectural comparison (printed at the end of Cell 19):**

```
Baseline  (max=20, pinning=False): refund_completed=True
Failure   (max= 3, pinning=False): refund_completed=False  ← AMNESIAC PIVOT
Fixed     (max= 3, pinning=True):  refund_completed=True
```

Same model. Same prompt content. Same conversation. Different memory configuration.

---

### Exercise B — Trigger the Episodic Blind Spot (Tier 2 failure)

The Episodic Blind Spot occurs when Tier 2 (episodic memory) is absent. Every query is treated as a first occurrence. A recurring issue is indistinguishable from a new one.

**To trigger the failure:**

In Cell 22, `tier2.disable()` is called before Session 2. Run the cell. The output will show:

```
🔴 EPISODIC BLIND SPOT
   Tier 2 disabled → 0 past episodes retrieved
   Agent treated recurrence as first occurrence
   Response is IDENTICAL to Session 1 — no recurrence flag
```

**To observe the fix:**

Cell 23 calls `tier2.enable()` before Session 2. Run the cell. The output will show:

```
✅ RECURRENCE DETECTED AND ESCALATED
   Tier 2 returned prior episode → agent flagged recurrence
   Immediate escalation instead of re-resolution attempt
```

**Deflection rate comparison across 10 sessions (Cell 24 — Level 5 evidence artifact):**

```
Tier 2 DISABLED:  deflection rate 100%, appropriate escalation 0%
Tier 2 ENABLED:   deflection rate 10%,  appropriate escalation 90%
```

The architecture with the lower deflection rate is performing better — it is catching failures the other configuration misses entirely.

---

## Mandatory Human Decision Node

Cell 7 contains the required Human Decision Node. The AI scaffold proposes embedding raw chat transcripts into Tier 2. The node documents the rejection:

```python
HUMAN_DECISION = {
    "accepted": False,
    "reason": "Raw transcripts create a Deferred Decision Hazard. "
              "PII embedded in a vector store is practically un-deletable "
              "without full index reconstruction. Samsung case: no write gate "
              "= permanent data exit.",
    "modification": "Insert SanitizationNode(redact_pii=True) between "
                    "Tier 1 capture and Tier 2 write."
}
```

This decision is architectural, not technical. The agent cannot make it because the input is the organization's legal posture and deletion obligations — neither of which is in any training corpus or knowledge base the agent can retrieve.

---

## Five-Tier Memory Stack

| Tier | Name | Storage | Failure if Absent | Internal Defect |
|------|------|---------|-------------------|-----------------|
| 1 | Working Memory | Context window (RAM) | Amnesiac Pivot | Context overflow / Lost in the Middle |
| 2 | Episodic Memory | Vector DB, user-keyed | Episodic Blind Spot | Scope collapse if user isolation fails |
| 3 | Semantic Memory | Knowledge graph + relational DB | No org-level context | Schema drift; unchecked write corruption |
| 4 | Procedural Memory | DAG rule engine | Planner Loop | Stale DAGs if procedures change without redeployment |
| 5 | Archival Memory | Vector DB + document store | Fabricated output; no citation (Air Canada failure mode) | 24-hr stale window; poor precision for short queries |

---

## Bloom's Taxonomy Evidence Artifacts

| Level | Exercise | Artifact |
|-------|----------|----------|
| 4 — Analyzing | Exercise A, Cell 18 | Annotated eviction log identifying the turn at which the goal token was evicted |
| 5 — Evaluating | Exercise B, Cell 24 | Deflection rate comparison across 10 sessions with written diagnosis |
| 6 — Creating | Exercise C (Chapter, p. final section) | Written HIPAA architecture specification — applies Tetrahedron to novel constrained system |

---

## Key References

- Liu, N. et al. (2023). *Lost in the Middle: How Language Models Use Long Contexts.* arXiv:2307.03172
- Moffatt v. Air Canada, BC Civil Resolution Tribunal, Case No. SC-CC-00462808, February 2024
- Samsung bans employee use of AI like ChatGPT after data leak. Bloomberg / Reuters, May 2023
- How My AskAI Built Self-Improving Support Agents. Qdrant Case Study, 2025. https://qdrant.tech/blog/case-study-my-askai/

---

## Architectural Claim

> Memory is not a feature. Memory is the system. The Air Canada agent had a model. What it did not have was a retrieval architecture that grounded that model's outputs in verifiable sources. $812.02 in damages is the measurable cost of an absent Tier 5 citation constraint.
