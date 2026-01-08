# Vocabulary Pattern: Why New Words Don’t Stick

When you introduce **new words, invented characters, symbolic alphabets, or domain‑specific terminology** to an AI through RAG or local context, you quickly discover a frustrating pattern:

> The AI *seems* to understand your new vocabulary in the moment,  
> but it does not **retain**, **stabilize**, or **generalize** it.

This document explains why this happens, using examples similar to those explored on *spireason.neocities.org* and related experimental sites.

---

## 1. RAG does not create new vocabulary entries

Retrieval‑augmented generation (RAG) works by **feeding text into the model’s short-term context**, not by modifying its internal dictionary or tokenization. This means:

- New words are not added to the model’s vocabulary.
- New characters are not added to its tokenizer.
- New symbolic structures are not integrated into its grammar.

RAG can only *remind* the model of the new term while it is visible in the context window. Once the context shifts, the model falls back to its **pretrained linguistic habits**.

### What this looks like in practice

- A new term like `spiralcode` may be used correctly in one paragraph,  
  but in the next, it becomes `spiral code`, `spirocode`, or `spiricode`.

- A custom symbol like `⧉` may be recognized once,  
  but later replaced with `[]`, `<>`, or dropped entirely.

- A constructed alphabet may be reproduced for a few lines,  
  then collapse into random Unicode or Latin approximations.

This is not “forgetting” in the human sense — it is the model reverting to its **statistical comfort zone**.

---

## 2. What *is* remembered: common-language expressions

Even when new vocabulary fails to stabilize, the model often retains:

- the **meaning** you intended,
- the **relationships** between concepts,
- the **semantic patterns** expressed in ordinary language.

This is because the model is trained to generalize meaning through **natural-language structures**, not through arbitrary symbol systems.

### Example pattern

If you introduce:

> “In Spiral Reasoning, a *node* is called a *spire*, and a *connection* is called a *twine*.”

The model may later forget the words *spire* and *twine*, but it will still remember:

- there are “nodes” and “connections,”  
- they belong to a system called “Spiral Reasoning,”  
- they relate in a specific structural way.

So the model might say:

> “In your system, each node connects to others through a kind of linking structure.”

The **meaning survived**, but the **vocabulary did not**.

---

## 3. Local understanding vs. global instability

AI models can often use your new terms correctly **within a single answer**, because:

- the term is present in the prompt,
- the model can pattern-match locally,
- the context window anchors the new vocabulary.

But globally — across multiple messages, or across different parts of a long text — the new terms:

- drift,
- mutate,
- get replaced,
- or vanish entirely.

This is because the model has no mechanism to:

- store new tokens,
- create new embeddings,
- or update its internal grammar.

Only **fine-tuning** or **tokenizer extension** can do that.

---

## 4. Why invented alphabets and symbols fail most dramatically

Experiments like those on *spireason.neocities.org* show that:

- invented alphabets,
- symbolic grammars,
- custom glyphs,
- or nonstandard character sets

are the **least stable** forms of new vocabulary.

### Why?

Because the model’s tokenizer was never trained to treat them as meaningful units. They are:

- split into many sub-tokens,
- treated as noise,
- or normalized away.

So the model cannot:

- repeat them reliably,
- place them in consistent order,
- or integrate them into sentence structure.

This is why symbolic languages “fall apart” after a few lines.

---

## 5. Why fine-tuning is required for stable vocabulary

To make new vocabulary *stick*, you need:

- repeated examples,
- consistent usage,
- and training that modifies the model’s internal weights.

Fine-tuning allows the model to:

- internalize new terms,
- associate them with meanings,
- and use them across contexts without reminders.

RAG cannot do this.  
Local context cannot do this.  
Prompt engineering cannot do this.

Only training can.

---

## 6. Summary of the Vocabulary Pattern

**Minus:**  
New words, symbols, and invented alphabets do not enter the model’s internal vocabulary. They are unstable, easily lost, and not generalized.

**Plus:**  
The model still extracts and retains the *meaning* behind your new terms, as long as that meaning can be expressed in ordinary language.

**Implication:**  
If your project depends on stable new terminology, you must move beyond RAG and into **fine-tuning** or **tokenizer extension**.
