# 📘 Part 1 — Graphical Tools for Working With Anki and JSON  
*A practical guide to human‑friendly card editing and machine‑friendly dataset formats*

Before Q&A cards can be used for AI training or fine‑tuning, they must exist in a format that is both:

- **human‑compliant** → easy to read, verify, edit  
- **machine‑compliant** → structured, predictable, JSON‑friendly  

Anki sits firmly in the first category.  
JSON sits firmly in the second.

This chapter explains how graphical tools bridge the gap.

---

## 🟣 Understanding Anki as a Human‑Compliant Format

Anki is designed for **humans**, not machines.  
Its strengths come from:

- **visual verification** — you can see each card exactly as a learner would  
- **fields** — front, back, hints, tags, custom metadata  
- **templates** — card styling, HTML/CSS layouts  
- **collections** — decks, subdecks, note types  
- **media support** — images, audio, LaTeX  

This makes Anki ideal for:

- reviewing Q&A  
- validating correctness  
- spotting ambiguous or poorly phrased cards  
- organizing large collections  

But it also means:

- the `.apkg` format is a **SQLite bundle**, not a simple text file  
- the structure is optimized for spaced repetition, not AI training  
- exporting requires conversion  

Anki is the **gold standard for human verification**, but not for machine consumption.

---

## 🟣 Understanding JSON as a Machine‑Close Format

JSON is the opposite:

- simple  
- flat  
- predictable  
- easy to parse  
- universally supported by fine‑tuning frameworks  

JSON (or JSONL) is the preferred format for:

- LitGPT  
- Hugging Face  
- OpenAI fine‑tuning  
- Axolotl  
- OpenPipe  
- most custom training pipelines  

But JSON is **not** human‑friendly:

- no styling  
- no templates  
- no deck hierarchy  
- no media embedding  
- no visual preview  

JSON is the **compiled** version of your cards — efficient, but not pleasant to edit manually.

---

# 🟣 Tools That Work With Anki Files Graphically

Below are the tools that allow you to **import Anki decks**, edit them graphically, and export them into formats closer to JSON.

---

## ⭐ CrowdAnki (Anki plugin — graphical inside Anki)

CrowdAnki is the most important bridge between Anki and JSON.

### What it does:
- Adds a **graphical JSON importer/exporter** inside Anki  
- Exports decks as **structured JSON**  
- Preserves:
  - fields  
  - templates  
  - tags  
  - deck hierarchy  
- Allows version control (GitHub‑friendly)  

### Why it matters:
CrowdAnki JSON is the **cleanest representation** of Anki data outside Anki itself.  
It is the closest thing to a “source code” format for flashcards.

### Limitations:
- Still requires Anki as the GUI  
- JSON is not optimized for fine‑tuning (needs transformation)

---

## ⭐ AnkiHub (web GUI)

AnkiHub is a cloud‑based graphical editor for Anki decks.

### What it does:
- Imports `.apkg`  
- Lets you edit cards in a web interface  
- Syncs changes back to Anki  
- Exports CSV (convertible to JSON)  
- Supports collaborative editing  

### Why it matters:
It is the only **web‑native graphical editor** for Anki decks.

### Limitations:
- JSON export is indirect  
- Subscription required for full features  

---

## ⭐ RemNote (imports Anki, exports JSON‑like formats)

RemNote is a flashcard‑first knowledge system.

### What it does:
- Imports Anki decks  
- Converts them into RemNote’s structured format  
- Exports:
  - JSON‑like files  
  - Markdown  
  - CSV  

### Why it matters:
RemNote’s export formats are **cleaner and more structured** than Anki’s native exports.

### Limitations:
- Not a perfect 1:1 mapping with Anki  
- Some template information is lost  

---

## ⭐ Memrise / Kitsun / Mochi (partial support)

These tools can import Anki decks and export:

- CSV  
- TSV  
- Markdown  
- JSON (in some cases)

They are graphical flashcard editors, but:

- support varies  
- templates and styling are often lost  
- media handling is inconsistent  

These are useful for **quick conversions**, not for full‑fidelity editing.

---

## ⭐ AnkiWeb + third‑party converters

AnkiWeb itself does not export JSON, but you can:

- export `.apkg`  
- use graphical converters (community tools)  
- convert to JSON or CSV  

This is the least polished option, but still viable.

---

# 🟣 Part 1B — Converting Anki to JSON: What Is Lost, What Is Gained

When converting Anki → JSON, you are effectively moving from:

**a rich, human‑oriented format**  
→  
**a flat, machine‑oriented format**

### What is preserved:
- front/back text  
- tags  
- basic fields  
- deck names  
- card content  

### What is partially lost:
- templates (HTML/CSS)  
- styling  
- media references  
- scheduling metadata  
- card type logic (cloze, multi‑card notes)  

### What is fully lost:
- learning history  
- ease factors  
- review intervals  
- deck options  
- card styling behavior  

### Why this matters:
JSON becomes a **low‑level representation** of the cards — similar to compiled code.

It is perfect for:

- fine‑tuning  
- dataset preprocessing  
- programmatic manipulation  

But not ideal for:

- human review  
- visual editing  
- pedagogical refinement  

This is why the workflow is usually:

**Anki (human editing) → JSON (machine training)**

---

# 📘 Part 2 — Graphical Tools to Postprocess JSON/Q&A Datasets

Once cards are in JSON or CSV, you can use graphical tools to inspect, clean, and prepare them for fine‑tuning.

---

## ⭐ Datasette + Datasette‑Lite

Datasette is a **graphical browser** for structured datasets.

### What it does:
- Loads JSON, CSV, SQLite  
- Lets you browse Q&A pairs  
- Provides filtering, sorting, searching  
- Runs locally or in the browser  

### Why it matters:
Datasette is ideal for **visual inspection** of large datasets.

### Limitations:
- Not an editor  
- No fine‑tuning integration  

---

## ⭐ Label Studio

Label Studio is a **graphical annotation tool**.

### What it does:
- Imports JSON  
- Lets you edit Q&A pairs  
- Supports text generation tasks  
- Exports JSONL  

### Why it matters:
It is the closest thing to a **graphical dataset editor** for fine‑tuning.

---

## ⭐ OpenPipe Studio

OpenPipe Studio is a cloud‑based dataset editor.

### What it does:
- Imports JSONL  
- Lets you edit prompts and completions  
- Validates datasets  
- Exports fine‑tuning‑ready JSONL  

### Why it matters:
It is the most polished **graphical fine‑tuning dataset editor** available today.

### Limitation:
- It fine‑tunes its own models, not LitGPT.

---

## ⭐ Hugging Face Datasets Viewer

The Hugging Face web interface can:

- load JSON  
- display Q&A pairs  
- allow browsing and filtering  
- export in multiple formats  

### Limitation:
- Not an editor  
- No Anki import  

---

# 📘 Part 3a — Are There Graphical Fine‑Tuners?

Short answer: **not really**.

There is no graphical tool that:

- imports Anki  
- imports JSON  
- edits Q&A  
- and directly triggers LitGPT fine‑tuning  

But two tools come close.

---

## ⭐ OpenPipe Studio

- graphical dataset editor  
- graphical fine‑tuning  
- evaluation tools  

But it trains **its own models**, not LitGPT.

---

## ⭐ Hugging Face AutoTrain

- graphical fine‑tuning  
- dataset upload  
- model selection  

But again, it trains **Hugging Face models**, not LitGPT.

---

# 📘 Part 3b — Tools Often Confused With Fine‑Tuners (But Are Not)

These tools are useful, but they do **not** fine‑tune models.

---

## ⭐ Nomic Atlas — Embedding Explorer

Nomic Atlas visualizes **embeddings**:

- clusters  
- semantic neighborhoods  
- dataset structure  

It is excellent for:

- dataset analysis  
- cleaning  
- understanding topic distribution  

But it does not train models.

---

## ⭐ LM Studio (partial)

LM Studio is graphical, but:

- runs inference only  
- no fine‑tuning  
- useful for testing Q&A datasets  

---

## ⭐ Ollama + GUI frontends

Ollama is CLI‑based, but GUIs exist:

- Open WebUI  
- AnythingLLM  
- LibreChat  

These can:

- load models  
- test Q&A  
- serve datasets  

But none can fine‑tune.

---

# 🎯 Final Summary

### ✔️ Tools that graphically handle Anki → JSON  
CrowdAnki, AnkiHub, RemNote, Mochi, Kitsun, Memrise, converters

### ✔️ Tools that graphically postprocess JSON  
Datasette, Label Studio, OpenPipe Studio, HF Viewer

### ✔️ Tools that graphically fine‑tune  
None that work with LitGPT  
Closest: OpenPipe Studio, HF AutoTrain

### ✔️ Tools that help analyze or test  
Nomic Atlas, LM Studio, Ollama GUIs

