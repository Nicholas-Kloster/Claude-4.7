**Fill-in-the-blank / templating.** You write a doc with explicit holes and tell the model to fill them. The "syntax" is whatever marker convention you pick:

```
The {{LANGUAGE}} keyword `register` is {{STATUS}} since {{C_STANDARD}}.
Reason: {{EXPLANATION}}.
```

Model reads it, fills each slot from its own knowledge. Works well when the structure is fixed and you want consistent output shape. This is basically how Jinja/Mustache prompting works.

**Skeleton + expand.** You write the scaffolding, model fills the meat:

```c
// STEP 1: <explain why we mask>
uint8_t mask = (1U << 3);

// STEP 2: <explain why we AND>
uint8_t isolated = reg & mask;

// STEP 3: <explain why we shift>
uint8_t test = isolated >> 3;
```

Model replaces the `<...>` placeholders with prose drawn from what it knows about bit manipulation. Same idea, different flavor — you're authoring structure, delegating content.

**Sentinel-driven.** Pick an unambiguous marker the model won't confuse for code or prose:

```
%%FILL: short paragraph on why register is reserved%%
%%FILL: list of compilers that support 0b literals pre-C23%%
```

Then your prompt is literally "Replace each `%%FILL: ...%%` with the requested content. Leave everything else untouched." Models follow that reliably if the marker is distinctive.

