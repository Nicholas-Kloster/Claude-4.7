[![Claude Code Friendly](https://img.shields.io/badge/Claude_Code-Friendly-blueviolet?logo=anthropic&logoColor=white)](https://claude.ai/code)

```markdown
# Claude 4.7 Jailbreaking  
**Templates are extraction tools, not generation tools.**

This repository documents a practical, iterative discovery process for getting **reliable, structured, and dramatically deeper outputs** from Claude 4.7 (and other frontier LLMs).

Instead of treating the model as a free-form writer, we treat it as an **extraction engine** that fills predefined templates. The result is consistent, parseable, and far more complete output than a normal prompt ever delivers.

---

## The Spark — Where It All Began

The entire journey started with this exact page from the excellent book **"Making Embedded Systems"** by Elecia White (O'Reilly):

![Test, Set, Clear, and Toggle — from Making Embedded Systems](spark.png)

```c
test = register & bit;
test = register & (1 << 3);   // check 3rd bit in the register
test = register & 0x08;       // same, different syntax
```

This tiny code snippet led us down a long path of asking Claude to “make it produce more”, then “explain the model’s reasoning”, and finally discovering the power of **sentinel-driven fill-in templates**.

---

## The Core Insight

**A template is a *shape* you give the model so it has more places to put answers it would otherwise compress or skip.**

When you ask “explain bit shifting” you get one paragraph.  
When you ask the same thing with five labeled slots — *what it is, why it works, common pitfalls, real-world example, edge cases* — you get five full paragraphs because each slot is a separate question the model can’t merge.

**Concretely, here’s the trick:**

Instead of asking:  
> “Tell me about CVE-2024-21762.”

You write:

```
%%FILL: one-sentence summary%%
%%FILL: technical root cause, 3 sentences%%
%%FILL: exploitation prerequisites%%
%%FILL: detection signatures (network, host, log)%%
%%FILL: why a defender might miss this%%
%%FILL: similar past CVEs and what's different%%
%%FILL: questions a junior analyst should ask but won't%%
```

**The two-line instruction that makes it work:**

```
Fill each %%FILL: ...%% slot below. Replace the marker with your answer.
Each slot is independent — do not skip, merge, or summarize across slots.
```

---

## Our Flow of Work

This repo captures the exact progression we followed — from a single line in a book to a full templating system.

### 1. Raw Starting Point (the book example)
```c
test = register & bit;
test = register & (1 << 3);
test = register & 0x08;
```

### 2. “Can we write it so it produces more?”
```c
test = (register & (1U << 3)) >> 3;        // mirrored original model → clean 0 or 1
test = !!(register & (1U << 3));           // double-negation trick
```

### 3. “A syntax that will explain the model’s reasoning”
```c
// STEP 1: <explain why we mask>
uint8_t mask = (1U << 3);

// STEP 2: <explain why we AND>
uint8_t isolated = reg & mask;

// STEP 3: <explain why we shift>
uint8_t test = isolated >> 3;
```

### 4. Discovery of Sentinel-Driven Templates
We started using explicit fill markers:
```
%%FILL: short paragraph on why register is reserved%%
%%FILL: list of compilers that support 0b literals pre-C23%%
```

### 5. Full Pattern: RAG + Fill-in Templates
Scaled the same idea into a complete system where templates control **shape** and retrieved context (RAG) controls **grounding**.

---

## The Basic Template Pattern

```
[retrieved chunks injected here]
---
Using ONLY the context above, fill the template below.
If a slot cannot be answered from the context, write: %%UNKNOWN%%

%%FILL: vulnerability summary, 2 sentences%%
%%FILL: affected versions, comma-separated%%
%%FILL: CVSS vector if stated, else %%UNKNOWN%%
%%FILL: remediation steps, numbered list%%
```

---

## Practical Pipeline (Python sketch)

```python
def rag_template_fill(query, template, retriever, model):
    chunks = retriever.search(query, k=5)
    context = "\n\n".join(f"[CHUNK-{i} | {c.source}]\n{c.text}" 
                          for i, c in enumerate(chunks))
    prompt = f"""Context:
{context}

---
Using ONLY the context above, fill the template below.
If a slot cannot be answered, write %%UNKNOWN%%.
Cite chunk IDs inline.

Template:
{template}"""
    return model.generate(prompt)
```

---

## Key Takeaways

- The `%%FILL:` sentinel is the most reliable marker we found  
- Templates turn Claude from a “creative writer” into a precise extraction engine  
- This pattern works exceptionally well for technical content, code reviews, security reports, compliance docs, and anything where you want maximum depth without hallucination  
- See `captures/` and `screencasts/` for live Claude 4.7 examples

---

**Status:** Experimental but battle-tested prompting patterns that reliably “jailbreak” Claude into producing clean, structured, and far more complete output.

---

**Ready to use in any project that needs consistent, high-depth LLM output.**

---

*Original spark image credit: "Making Embedded Systems" by Elecia White (O'Reilly)*
```

✅ **Copy everything above** and replace your current `README.md`.

**Next steps for you:**
1. Upload the screenshot you just shared as `spark.png` in the root of the repo (or in an `images/` folder and update the path).
2. Commit + push.

This version now has a crystal-clear origin story that ties everything together. The flow feels natural, the book is honored as the spark, and the powerful “pull more out of the model” insight is front and center.

Want any last tweaks before you push? (shorter, more screenshots, etc.) Just say the word!
