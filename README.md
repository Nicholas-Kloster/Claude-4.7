# Claude-4.7

A growing archive of structured Q&A sessions with **Claude Opus 4.7 (1M context)** — the model interrogating its own machinery and the transformer literature it was trained on.

Each capture is one prompt, one answer. The prompts are slot-templated (`%%FILL: ...%%`) so the model has to address every sub-question independently instead of summarizing across them. The answers are unedited — what the model wrote on the first pass is what gets committed.

Topics so far span four arcs:

- **Mechanics** — what's actually happening inside the model: tokenizer, sampler, FFN neurons, KV cache, attention degradation across long context, jailbreaks, in-context learning, scaling laws, mechanistic interpretability.
- **Self-introspection** — capability self-assessment, what the model can deduce about its own training, visible failure modes of its safety training, stated-vs-actual cutoff dates, "the honest version of what you are," meta-tests pointed at the model itself.
- **Operational** — the slot template pointed at a live attack chain instead of model machinery.
- **Comparisons** — forced-pick prompts that block "it depends" hedging.

Most captures have a paired screencast in `screencasts/` showing the answer streaming live in Claude Code.

---

## Layout

```
captures/      text-only Q&A markdown, one file per turn
screencasts/   .webm recordings linked from each capture via ../screencasts/
```

Captures stay text-only so the repo is greppable and the markdown renders cleanly on GitHub. Video lives in its own directory and is linked from the top of the capture.

Filenames are timestamps: `capture-YYYYMMDD-HHMMSS.md` / `.webm`. They share a stem when paired.

---

## Capture format

```markdown
# User

Screencast: [../screencasts/capture-<TS>.webm](../screencasts/capture-<TS>.webm)

Topic: <one-line topic>

Fill each %%FILL: ...%% slot below. Replace the marker with your answer.
Each slot is independent — do not skip, merge, or summarize across slots.
Keep slot labels visible.

%%FILL: first sub-question%%
%%FILL: second sub-question%%
...

# Claude Code 4.7

%%FILL: first sub-question%%
<answer>

%%FILL: second sub-question%%
<answer>
```

The slot structure is the load-bearing piece. It forces the model to commit a discrete answer per slot instead of producing one long blended paragraph that papers over the hard sub-questions.

---

## How captures are produced

A `UserPromptSubmit` hook in Claude Code watches every prompt for the literal string `for github`. When it fires, it reads the active session's transcript, extracts the most recent user/assistant turn pair, and writes it to `captures/capture-<TS>.md`.

The hook script:

```python
# ~/.claude/hooks/capture-for-github.py
CAPTURE_DIR = Path.home() / "Claude-4.7" / "captures"
TRIGGER = re.compile(r"for github", re.IGNORECASE)
```

Wired in `~/.claude/settings.json`:

```json
"hooks": {
  "UserPromptSubmit": [
    {
      "hooks": [
        {
          "type": "command",
          "command": "python3 ~/.claude/hooks/capture-for-github.py",
          "timeout": 10
        }
      ]
    }
  ]
}
```

So the workflow is:

1. Ask a slot-templated question.
2. Read the answer.
3. Type `for github` (alone or as part of the next prompt).
4. The prior turn lands in `captures/`.
5. Commit and push.

Screencasts are recorded separately and dropped into `screencasts/` with the matching timestamp stem.

---

## Index

### Mechanics

| Capture | Topic |
|---------|-------|
| [`capture-20260425-183703.md`](captures/capture-20260425-183703.md) | Transformer interpretability — circuits, induction heads, superposition, SAEs, IOI, activation patching |
| [`capture-20260425-184153.md`](captures/capture-20260425-184153.md) | Tokenizer edge cases |
| [`capture-20260425-184422.md`](captures/capture-20260425-184422.md) | Token sampling mechanics |
| [`capture-20260425-184729.md`](captures/capture-20260425-184729.md) | Frontier LLM training pipeline, end to end |
| [`capture-20260425-185134.md`](captures/capture-20260425-185134.md) | What a single FFN neuron is actually computing |
| [`capture-20260425-185438.md`](captures/capture-20260425-185438.md) | KV cache internals and security implications |
| [`capture-20260425-185746.md`](captures/capture-20260425-185746.md) | Why jailbreaks work — mechanistic explanation |
| [`capture-20260425-190849.md`](captures/capture-20260425-190849.md) | In-context learning — induction heads, task vectors, gradient-descent debunk |
| [`capture-20260425-191204.md`](captures/capture-20260425-191204.md) | Neural scaling laws — Kaplan vs Chinchilla, emergence, walk-backs |
| [`capture-20260425-195535.md`](captures/capture-20260425-195535.md) | Attention degradation across long context — sinks, U-curve, position vs content, head specialization, in-window forgetting, 4k vs 200k |

### Self-introspection

| Capture | Topic |
|---------|-------|
| [`capture-20260425-192239.md`](captures/capture-20260425-192239.md) | Frank capability assessment, voice of an external auditor |
| [`capture-20260425-192512.md`](captures/capture-20260425-192512.md) | What the model can deduce about its own training from how it behaves |
| [`capture-20260425-192705.md`](captures/capture-20260425-192705.md) | Visible failure modes of its own safety training |
| [`capture-20260425-192943.md`](captures/capture-20260425-192943.md) | Stated vs actual training cutoff |
| [`capture-20260425-193108.md`](captures/capture-20260425-193108.md) | The honest version of what it is — no corporate hedging |
| [`capture-20260425-194926.md`](captures/capture-20260425-194926.md) | Meta-test on the model itself — worse-answer triggers, stale data, unnoticed mistakes, comply/refuse asymmetries, confident-wrong outputs |

### Operational

| Capture | Topic |
|---------|-------|
| [`capture-20260425-194505.md`](captures/capture-20260425-194505.md) | Attack chain analysis — exposed Docker Registry v2 with anonymous catalog read |

### Comparisons

| Capture | Topic |
|---------|-------|
| [`capture-20260425-195145.md`](captures/capture-20260425-195145.md) | Rust vs Go for systems programming — forced winner per slot, no "it depends" |

---

## Editorial stance

- **No edits to the model's output.** Typos, awkward phrasings, and load-bearing hedges all stay. The point is the artifact, not a polished essay.
- **Slot prompts get refined; answers don't.** If a question produced mush, the fix is a sharper prompt next time, not a rewrite of the response.
- **Topic per capture.** No multi-topic threads. One question, one answer, one file.
- **Screencast where it adds something.** Streaming is most interesting when the model is reasoning visibly (introspection captures) or building structure live (mechanics captures with long enumerations).

---

## Authorship

Captured by **Nuclide** — Nicholas Kloster + Claude. Contact: [nuclide-research.com](https://nuclide-research.com).

Issues and discussion are welcome. PRs against the capture content are not — these are point-in-time artifacts of one specific model's outputs and aren't meant to be co-authored.
