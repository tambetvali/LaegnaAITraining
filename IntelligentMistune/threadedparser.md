# 1. What This Architecture Is Doing

This architecture turns a **single Markdown file** into **three different layers of structured training data**, all extracted simultaneously:

1. **Block‑level training items**  
   Triggered by: `[/BLOCKREAD/]`  
   Purpose: Teach an AI patterns of “Q = ID, A = block content”.

2. **Chapter‑level training items**  
   Triggered by: `[/CHAPTREEREAD/]`  
   Purpose: Teach an AI how chapter hierarchy maps to full explanations.

3. **Q&A cards**  
   Triggered by: `[/Q]... [Q/]` and `[/A]... [A/]`  
   Purpose: Generate Anki cards and supervised training pairs.

All three parsers operate independently. A single Markdown file can therefore produce **multiple training datasets** at once.

The file format is **flexible**:
- If a parser sees its trigger tag, it activates.
- If not, it ignores that part.
- The Q&A parser always runs, even if the others are absent.

So one document becomes a **multi‑purpose training source** that can evolve without breaking older parsers.

---

# 2. What the Initial Markdown Looks Like

Below is the Markdown file used in the example, written so you can copy it as text:

\`\`\`markdown
\# [/T]Laegna AI Training Overview[T/]

Welcome to the LaegnaAITraining demo. In this document, we will see how block and chapter readers work. [/BLOCKREAD/]

\## [/T]Block Reader Explanation[T/]

The block reader is triggered when we include the tag [/BLOCKREAD/] in the text.

[/Q]What does the block reader do?[Q/]
[/A]It processes each block (like a title, paragraph, or code block) and stores it as a Q/A-style training item, where Q is a unique identifier (file and block index) and A is the block content.[A/]

[/BLOCKREAD/]

\### [/T]Sample code block[T/]

\`\`\`python
def hello():
    print("Hello, Laegna!")
\`\`\`

This code block is another example that the block reader will capture. [/BLOCKREAD/]

\## [/T]Chapter Tree Reader Explanation[T/]

The chapter tree reader is triggered by the tag [/CHAPTREEREAD/]. It captures the entire content of a chapter, together with the chapter hierarchy (tree) up to that point.

[/Q]Why is the chapter tree representation helpful?[Q/]
[/A]Because the model can see how high-level topics subdivide into subtopics and detailed explanations.[A/]

[/CHAPTREEREAD/]

\## [/T]Q & A Usage[T/]

This chapter focuses on how Q and A tags are used to generate Anki cards and training data.

[/Q]How are Q & A tags transformed into Anki cards?[Q/]
[/A]Each [/Q]... [Q/] section becomes the front of a card, and each [/A]... [A/] section becomes the back. The parent chapter and higher-level headings form a context path that appears on the card, making recall easier and more structured.[A/]

[/CHAPTREEREAD/]
\`\`\`

---

# 3. The Three Parsers

Each parser has its own trigger tag and produces its own type of training data from the same Markdown.

---

## 3.1. Parser 1 — Block Reader (`[/BLOCKREAD/]`)

### What it does

- Looks for `[/BLOCKREAD/]` markers in the document.
- Around those regions, each **block** (paragraph, heading, code block) is captured as a training item.
- For each captured block it creates:
  - **Q** = `file_id#blockN` (e.g. `example_file#block3`)
  - **A** = the block’s raw content (text or code).

This gives you a structured pattern like:

\`\`\`text
Q=example_file#block1
A=Welcome to the LaegnaAITraining demo...

Q=example_file#block2
A=The block reader is triggered when we include...
\`\`\`

The model sees the repeated schema `Q=something`, `A=content`, and can learn relationships between different block types (titles, paragraphs, code).

### Example cards produced (first 3)

\`\`\`text
1) Q=example_file#block1
   A=Welcome to the LaegnaAITraining demo. In this document, we will see how block and chapter readers work.

2) Q=example_file#block2
   A=The block reader is triggered when we include the tag [/BLOCKREAD/] in the text.

3) Q=example_file#block3
   A=def hello():
       print("Hello, Laegna!")
\`\`\`

### Total block items

For the example document, you might get **about 5–8 block items**, depending on how you segment paragraphs and code blocks.

---

## 3.2. Parser 2 — Chapter Tree Reader (`[/CHAPTREEREAD/]`)

### What it does

- Tracks headings `#`, `##`, `###` as a **chapter tree**. For example:

  - `# Laegna AI Training Overview`
  - `## Chapter Tree Reader Explanation`

  becomes:

  \`\`\`text
  ["Laegna AI Training Overview", "Chapter Tree Reader Explanation"]
  \`\`\`

- When it encounters `[/CHAPTREEREAD/]`, it takes:

  - The **current chapter ID** (like `2` or `2.1`),
  - The **chapter tree** (list of titles),
  - The **full chapter content** (all text and code under that heading),

  and stores this as a single training item.

This links high‑level structure (table‑of‑contents style) with the actual explanatory text.

### Example chapter items (first 2)

\`\`\`text
1) CHAPTER_ID=2
   FILE_ID=example_file
   TREE=["Laegna AI Training Overview", "Chapter Tree Reader Explanation"]
   CONTENT="The chapter tree reader is triggered by the tag [/CHAPTREEREAD/]. It captures the entire content of a chapter, together with the chapter hierarchy (tree) up to that point.

   Why is the chapter tree representation helpful? ..."

2) CHAPTER_ID=3
   FILE_ID=example_file
   TREE=["Laegna AI Training Overview", "Q & A Usage"]
   CONTENT="This chapter focuses on how Q and A tags are used to generate Anki cards and training data.

   How are Q & A tags transformed into Anki cards? ..."
\`\`\`

### Total chapter items

In this example there are **2 chapter items** (one for “Chapter Tree Reader Explanation”, one for “Q & A Usage”).

---

## 3.3. Parser 3 — Q&A Tag Parser (`[/Q]`, `[/A]`, `[/T]`)

### What it does

- Scans the Markdown for pairs of:

  - `[/Q]... [Q/]` → question text,
  - `[/A]... [A/]` → answer text.

- Uses the current **chapter tree** (built from `#`, `##`, `###` and `[ /T ]...[T/]` tags) as **context**.
- For each Q/A pair, creates an **Anki card**:

  - **Front**: context path (chapter hierarchy) + question.
  - **Back**: answer.

These are both:

- Ready for human learning (Anki).
- Good supervised pairs for model fine‑tuning.

### Example Q&A cards (first 3)

\`\`\`text
1) FRONT:
   Laegna AI Training Overview / Block Reader Explanation

   Q: What does the block reader do?

   BACK:
   A: It processes each block (like a title, paragraph, or code block) and stores it as a Q/A-style training item...

2) FRONT:
   Laegna AI Training Overview / Chapter Tree Reader Explanation

   Q: Why is the chapter tree representation helpful?

   BACK:
   A: Because the model can see how high-level topics subdivide into subtopics and detailed explanations...

3) FRONT:
   Laegna AI Training Overview / Q & A Usage

   Q: How are Q & A tags transformed into Anki cards?

   BACK:
   A: Each [/Q]... [Q/] section becomes the front of a card, and each [/A]... [A/] section becomes the back...
\`\`\`

### Total Q&A cards

In this example there are **3 Q&A cards** (one in each of the three main sections).

---

# 4. How Flexible Is This File Type?

The file type is intentionally **very flexible**.

### Multiple parser “versions”

You effectively support **multiple “versions” of card generation** from the same file:

- **Version A**: Use only Q&A cards (ignore `[/BLOCKREAD/]` and `[/CHAPTREEREAD/]`).
- **Version B**: Use block cards + Q&A.
- **Version C**: Use chapter cards + Q&A.
- **Version D**: Use all three.

You can choose which parsers to run at processing time without changing the Markdown.

### Orthogonal triggers

Each parser reacts only to **its own trigger tags**:

- `[/BLOCKREAD/]` → block reader.
- `[/CHAPTREEREAD/]` → chapter tree reader.
- `[/Q]`, `[/A]`, `[/T]` → Q&A / context parser.

This means you can:

- Add more tags and parsers later (e.g. glossary, examples, exercises).
- Ignore some tags in certain pipelines.
- Keep backward compatibility as the spec grows.

### Always‑on Q&A

Even if you remove all `[/BLOCKREAD/]` and `[/CHAPTREEREAD/]` tags, the Q&A parser still works—  
as long as your Markdown uses `[/Q]` and `[/A]`, you get Q&A cards.

---

# 5. Summary

From **one Markdown document**, your architecture produces:

| Parser               | Trigger              | Output                                | Purpose                          |
|----------------------|----------------------|----------------------------------------|----------------------------------|
| Block Reader         | `[/BLOCKREAD/]`      | `Q=ID, A=block content`               | Pattern learning on local blocks |
| Chapter Tree Reader  | `[/CHAPTREEREAD/]`   | Chapter tree + full chapter content   | Structural / hierarchical logic  |
| Q&A Tag Parser       | `[/Q]`, `[/A]`, `[/T]` | Anki-ready Q&A cards with context    | Human study + supervised pairs   |

The Markdown file itself remains:

- **Human‑readable** (it’s still a normal document),
- **Machine‑parsable** (tags give structure),
- **Extensible** (new tags/parsers can be added),
- **Versionable** (pipelines can pick and choose which parsers to run).

So one authoring surface feeds both:

- **Humans** (via HTML rendering + Anki cards), and  
- **AI models** (via structured block/chapter/Q&A corpora).
