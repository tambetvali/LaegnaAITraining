# LaegnaAITraining end‑to‑end example with Mistune, Flask, Anki, and LitGPT

This file walks through:

- **How** a Markdown document is processed by **Mistune** with three custom extensions.
- **How** those extensions:
  - Extract per‑block training data (`[/BLOCKREAD/]`)
  - Extract per‑chapter training data with chapter tree (`[/CHAPTREEREAD/]`)
  - Parse Q/A tags (`[/Q]`, `[/A]`, `[/T]`)
- **How** to:
  - Turn the extracted Q/A info into **Anki cards** (JSON + simple TSV/text).
  - Serve the processed HTML via **Flask**, download Anki cards, and offer a **LitGPT training link**.
  - Run everything as a **single‑file server**.
- **How** to install dependencies and run on Linux, with notes for Windows and macOS.

Below are code and configuration snippets. Each code snippet is marked with `=>\`\`\`language` as you requested, and is indented by four spaces so the outer Markdown code block stays intact.

---

## 1. Conceptual overview: how Mistune processes the Markdown

We will use **Mistune** as a Markdown parser with a custom renderer and plugins:

- **Core idea:**
  - Mistune parses Markdown into blocks (paragraphs, headings, lists, code blocks, etc.) and inlines.
  - Our **first extension** hooks into **block rendering** when we encounter a special tag `[/BLOCKREAD/]` in the text. For each code block we process, we:
    - Assign a **Q identifier**: `Q=<file_id>#<block_index>`.
    - Extract **A**: the **content of that block**.
    - Store it in a global or request‑local array, as if we were "teaching" an AI on consistent QA‑style items.
  - Because the format is **structured and repetitive** (`Q=...; A=...`), it lends itself well to pattern detection, prompting, and training. A model sees:
    - Many examples of `Q=ID; A=Content`
    - Patterns of content types, e.g., headings, paragraphs, code blocks.
    - This can be used to:
      - Pretrain a model to link **structure** and **content**.
      - Later prompt the model to answer "given `Q=...` what is `A`?" or "generate `A` for a new `Q`".

- **Second extension**:
  - Reacts to the `[/CHAPTREEREAD/]` tag.
  - Keeps a **chapter tree**: list of headings (`#`, `##`, `###`) seen so far.
  - For each chapter (i.e., a top‑level or second‑level heading region), when `[/CHAPTREEREAD/]` is encountered:
    - Constructs a descriptor:
      - `file_id` (session or file name).
      - `chapter_id` (e.g., "1", "1.2", etc.).
      - **chapter tree** up to current level.
    - Expects the **full chapter content** as "A".
  - This is **highly explanative** for an AI because:
    - Each training item contains:
      - The hierarchical context (chapter tree).
      - The entire chapter’s explanation.
    - The model learns to:
      - Map hierarchical structures (like a table of contents) to full explanations.
      - Use **chapter context** when generating or summarizing content later.

- **Third extension**:
  - Recognizes:
    - `[/T]... [T/]` as **topic/title inside a heading**.
    - `[/Q]... [Q/]` as **question text**.
    - `[/A]... [A/]` as **answer text**.
  - This extension will:
    - Record questions and answers per chapter.
    - Link them via the **parent chapter**.
  - We will:
    - Use this to generate **Anki cards**:
      - Front: Q text (plus parent chapter/topic context).
      - Back: A text (and maybe additional notes).

---

## 2. Overall one‑file Python server structure

We will implement a single Python file, e.g. `laegna_server.py`, that:

1. Implements three Mistune extensions:
   - **BlockReadExtension** for `[/BLOCKREAD/]`.
   - **ChapTreeReadExtension** for `[/CHAPTREEREAD/]`.
   - **QATagExtension** for `[/Q]`, `[/A]`, `[/T]`.

2. Reads an **example Markdown document** from a Python variable.

3. Runs Mistune with our extensions to:
   - Produce **HTML**.
   - Collect **block‑level training items**.
   - Collect **chapter‑level training items**.
   - Collect **Q/A card candidates**.

4. Generates **Anki card data**:
   - JSON (good for machines).
   - A simple TSV or text format (easiest to import into Anki).

5. Exposes a **Flask** web server with:
   - `/` – main page displaying HTML and some debugging info.
   - `/anki_cards.json` – download of JSON card data.
   - `/anki_cards.txt` – simple TSV for Anki import.
   - `/litgpt_train_cmd` – page showing how to train LitGPT on these cards.

6. Provides **LitGPT** training commands that:
   - Assume LitGPT is installed via `pip` or `conda`.
   - Use our generated Anki JSON or a transformed text file as training data.

---

## 3. Full Python implementation (single‑file server)

### 3.1. Dependencies and installation

You need Python 3.x installed. Then install dependencies.

**On Linux (Ubuntu‑like):**

    =>```bash
    # Update package index
    sudo apt update

    # (Optional) Install Python3 and virtualenv if not present
    sudo apt install -y python3 python3-venv python3-pip

    # Create and activate a virtual environment
    python3 -m venv .venv
    source .venv/bin/activate

    # Install needed Python packages
    pip install mistune flask litgpt

    # Optionally, install other tools you might need
    pip install tqdm datasets
    =>```

**On macOS (with Homebrew and system Python 3):**

    =>```bash
    # Install Python if you don’t have it
    brew install python

    python3 -m venv .venv
    source .venv/bin/activate

    pip install mistune flask litgpt
    =>```

**On Windows (PowerShell):**

    =>```powershell
    # Ensure Python installed and on PATH
    # Then:

    python -m venv .venv
    .\.venv\Scripts\Activate.ps1

    pip install mistune flask litgpt
    =>```

> Tip: Use a virtualenv to keep your environment clean and reproducible.

---

### 3.2. Single‑file implementation: `laegna_server.py`

Put all of this into `laegna_server.py`. Note: any internal triple backticks are escaped as `\`\`\`` so that this outer block doesn’t break.

    =>```python
    #!/usr/bin/env python
    import os
    import json
    from dataclasses import dataclass, asdict, field
    from typing import List, Dict, Any, Tuple

    from flask import Flask, Response, render_template_string, send_file, jsonify
    import mistune

    # ------------------------------
    # 1. Data models for collected info
    # ------------------------------

    @dataclass
    class BlockTrainingItem:
        q_id: str       # e.g., "file1#block3"
        content: str    # block content (A)

    @dataclass
    class ChapterTrainingItem:
        file_id: str
        chapter_id: str          # e.g., "1", "1.2"
        chapter_tree: List[str]  # list of chapter titles up to here
        content: str             # full chapter content

    @dataclass
    class AnkiCard:
        deck: str
        front: str
        back: str
        tags: List[str] = field(default_factory=list)


    # ------------------------------
    # 2. Global "session" storage for demo
    #    (In a real app you might have per-request or database)
    # ------------------------------

    class TrainingContext:
        def __init__(self, file_id: str):
            self.file_id = file_id
            self.block_items: List[BlockTrainingItem] = []
            self.chapter_items: List[ChapterTrainingItem] = []
            self.anki_cards: List[AnkiCard] = []
            # For chapter tracking
            self.chapter_stack: List[Tuple[int, str]] = []  # (level, title)
            self.current_chapter_id_stack: List[int] = []   # for numbering like 1, 1.1, etc.
            self.current_chapter_content: List[str] = []
            self.current_chapter_level: int = 0

        def start_chapter(self, level: int, title: str):
            # Flush previous chapter, if any
            self._flush_current_chapter()

            # Adjust chapter id stack based on level
            while len(self.current_chapter_id_stack) >= level:
                self.current_chapter_id_stack.pop()
            while len(self.current_chapter_id_stack) < level:
                self.current_chapter_id_stack.append(0)
            self.current_chapter_id_stack[-1] += 1

            # Update chapter stack
            while self.chapter_stack and self.chapter_stack[-1][0] >= level:
                self.chapter_stack.pop()
            self.chapter_stack.append((level, title))

            # Reset content buffer
            self.current_chapter_content = []
            self.current_chapter_level = level

        def append_to_current_chapter(self, text: str):
            self.current_chapter_content.append(text)

        def get_current_chapter_tree(self) -> List[str]:
            return [t for (lvl, t) in self.chapter_stack]

        def _flush_current_chapter(self):
            if self.current_chapter_content and self.chapter_stack:
                chapter_tree = self.get_current_chapter_tree()
                chapter_id = ".".join(str(n) for n in self.current_chapter_id_stack)
                item = ChapterTrainingItem(
                    file_id=self.file_id,
                    chapter_id=chapter_id,
                    chapter_tree=chapter_tree,
                    content="\n".join(self.current_chapter_content).strip()
                )
                self.chapter_items.append(item)
                self.current_chapter_content = []

        def finalize(self):
            self._flush_current_chapter()


    # ------------------------------
    # 3. Custom Mistune renderer + plugins
    # ------------------------------

    class CustomHTMLRenderer(mistune.HTMLRenderer):
        """
        We extend HTMLRenderer so we can hook into headings, paragraphs, and code blocks.
        We will use a TrainingContext instance to store extracted data.
        """
        def __init__(self, training_context: TrainingContext, *args, **kwargs):
            super().__init__(*args, **kwargs)
            self.tc = training_context
            self.block_counter = 0

        # ---- Heading ----
        def heading(self, text: str, level: int) -> str:
            # If the heading contains [/T]..[T/], we extract the topic
            topic = text
            if "[/T]" in text and "[T/]" in text:
                # Very simple parsing: everything between [/T] and [T/]
                start = text.find("[/T]") + len("[/T]")
                end = text.find("[T/]")
                topic = text[start:end].strip()
            # Track chapter structure
            self.tc.start_chapter(level, topic)
            # Also add heading text to chapter content
            self.tc.append_to_current_chapter(topic)
            # Render normal HTML heading
            return f'<h{level}>{mistune.escape(topic)}</h{level}>\n'

        # ---- Paragraph ----
        def paragraph(self, text: str) -> str:
            cleaned = text.replace('[/BLOCKREAD/]', '').replace('[/CHAPTREEREAD/]', '')
            # Append to chapter content
            self.tc.append_to_current_chapter(cleaned)
            # normal rendering
            return f'<p>{cleaned}</p>\n'

        # ---- Code Blocks ----
        def block_code(self, code: str, info: str | None = None) -> str:
            self.block_counter += 1
            q_id = f"{self.tc.file_id}#block{self.block_counter}"
            # Store block as training item
            self.tc.block_items.append(BlockTrainingItem(q_id=q_id, content=code))
            # Append to chapter content as well
            # Note: we escape backticks to keep surrounding Markdown stable
            self.tc.append_to_current_chapter(f"\\`\\`\\`{info or ''}\\n{code}\\n\\`\\`\\`")
            # Render html
            info_class = f' class="language-{info}"' if info else ''
            return f'<pre><code{info_class}>{mistune.escape(code)}</code></pre>\n'


    # ------------------------------
    # 4. Plugin for Q/A tags to generate Anki cards
    # ------------------------------

    def qa_tag_plugin(md: mistune.Markdown, training_context: TrainingContext):
        """
        Very lightweight plugin that post-processes the parsed Markdown text
        to find [/Q]... [Q/] and [/A]... [A/].
        We'll run this as a separate pass on the original Markdown.
        """

        def extract_qa_pairs(text: str) -> List[AnkiCard]:
            cards = []
            pos = 0
            while True:
                q_start = text.find("[/Q]", pos)
                if q_start == -1:
                    break
                q_end = text.find("[Q/]", q_start)
                if q_end == -1:
                    break
                question = text[q_start + len("[/Q]"):q_end].strip()

                a_start = text.find("[/A]", q_end)
                if a_start == -1:
                    break
                a_end = text.find("[A/]", a_start)
                if a_end == -1:
                    break
                answer = text[a_start + len("[/A]"):a_end].strip()

                pos = a_end + len("[A/]")

                # Use chapter tree as context if available:
                chapter_tree = training_context.get_current_chapter_tree()
                context_title = " / ".join(chapter_tree) if chapter_tree else "General"

                front = f"{context_title}\n\nQ: {question}"
                back = f"A: {answer}"

                cards.append(AnkiCard(
                    deck="LaegnaAITraining",
                    front=front,
                    back=back,
                    tags=["qa", "markdown", "laegna"]
                ))
            return cards

        # Attach a simple post-processing function to markdown object
        def parse_qa(text: str) -> None:
            cards = extract_qa_pairs(text)
            training_context.anki_cards.extend(cards)

        # We store this helper on the markdown instance for easiest use
        md.parse_qa = parse_qa  # type: ignore


    # ------------------------------
    # 5. Example Markdown content
    # ------------------------------

    EXAMPLE_MARKDOWN = r"""
    # [/T]Laegna AI Training Overview[T/]

    Welcome to the LaegnaAITraining demo. In this document, we will see how block and chapter readers work. [/BLOCKREAD/]

    ## [/T]Block Reader Explanation[T/]

    The block reader is triggered when we include the tag [/BLOCKREAD/] in the text.

    [/Q]What does the block reader do?[Q/]
    [/A]It processes each block (like a title, paragraph, or code block) and stores it as a Q/A-style training item, where Q is a unique identifier (file and block index) and A is the block content. This is useful for pattern-based learning because the model repeatedly sees the same schema: Q=ID, A=Content. Over time, the model can infer structure, context, and relationships between different types of blocks.[A/]

    [/BLOCKREAD/]

    ### [/T]Sample code block[T/]

    \`\`\`python
    def hello():
        print("Hello, Laegna!")
    \`\`\`

    This code block is another example that the block reader will capture. [/BLOCKREAD/]

    ## [/T]Chapter Tree Reader Explanation[T/]

    The chapter tree reader is triggered by the tag [/CHAPTREEREAD/]. It captures the entire content of a chapter, together with the chapter hierarchy (tree) up to that point.

    [/Q]Why is the chapter tree representation helpful?[Q/]
    [/A]Because the model can see how high-level topics subdivide into subtopics and detailed explanations. This maps a tree-like structure (like a table of contents) onto full explanatory text, which is highly useful when training the model to navigate topics, summarize sections, or generate context-aware answers.[A/]

    [/CHAPTREEREAD/]

    ## [/T]Q & A Usage[T/]

    This chapter focuses on how Q and A tags are used to generate Anki cards and training data.

    [/Q]How are Q & A tags transformed into Anki cards?[Q/]
    [/A]Each [/Q]... [Q/] section becomes the front of a card, and each [/A]... [A/] section becomes the back. The parent chapter and higher-level headings form a context path that appears on the card, making recall easier and more structured.[A/]

    [/CHAPTREEREAD/]
    """


    # ------------------------------
    # 6. Markdown processing function
    # ------------------------------

    def process_markdown(file_id: str, markdown_text: str) -> Tuple[str, TrainingContext]:
        tc = TrainingContext(file_id=file_id)
        renderer = CustomHTMLRenderer(training_context=tc)
        # Basic Markdown with our renderer
        md = mistune.create_markdown(renderer=renderer)

        # Render HTML (this populates tc.block_items and tc.chapter_items partially)
        html = md(markdown_text)
        # Finalize chapter items
        tc.finalize()

        # Attach QA plugin and parse QA from original text
        qa_tag_plugin(md, tc)
        md.parse_qa(markdown_text)

        return html, tc


    # ------------------------------
    # 7. Flask web app
    # ------------------------------

    app = Flask(__name__)

    # For simplicity, process the example markdown once at startup
    HTML_CONTENT, TRAINING_CONTEXT = process_markdown("example_file", EXAMPLE_MARKDOWN)


    @app.route("/")
    def index():
        # show html, plus small summary of collected items
        num_blocks = len(TRAINING_CONTEXT.block_items)
        num_chapters = len(TRAINING_CONTEXT.chapter_items)
        num_cards = len(TRAINING_CONTEXT.anki_cards)

        # Very simple template
        template = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>LaegnaAITraining Demo</title>
            <meta charset="utf-8" />
            <style>
                body { font-family: sans-serif; margin: 2rem; }
                pre { background: #f5f5f5; padding: 0.5rem; }
                code { font-family: monospace; }
                .summary { padding: 1rem; background: #eef; margin-bottom: 1rem; }
                .links a { margin-right: 1rem; }
            </style>
        </head>
        <body>
            <div class="summary">
                <h2>LaegnaAITraining Demo</h2>
                <p><strong>Blocks captured:</strong> {{ num_blocks }}</p>
                <p><strong>Chapters captured:</strong> {{ num_chapters }}</p>
                <p><strong>Anki cards:</strong> {{ num_cards }}</p>
                <div class="links">
                    <a href="/anki_cards.json">Download Anki JSON</a>
                    <a href="/anki_cards.txt">Download Anki TSV/Text</a>
                    <a href="/litgpt_train_cmd">LitGPT Training Instructions</a>
                </div>
            </div>
            <hr />
            <div class="content">
                {{ html_content | safe }}
            </div>
        </body>
        </html>
        """
        return render_template_string(template,
                                      html_content=HTML_CONTENT,
                                      num_blocks=num_blocks,
                                      num_chapters=num_chapters,
                                      num_cards=num_cards)


    @app.route("/anki_cards.json")
    def anki_cards_json():
        data = [asdict(card) for card in TRAINING_CONTEXT.anki_cards]
        return jsonify(data)


    @app.route("/anki_cards.txt")
    def anki_cards_txt():
        # Simple tab-separated format: deck \t front \t back \t tags
        lines = []
        for card in TRAINING_CONTEXT.anki_cards:
            tag_str = " ".join(card.tags)
            line = f"{card.deck}\t{card.front}\t{card.back}\t{tag_str}"
            lines.append(line)
        text = "\n".join(lines)
        return Response(text, mimetype="text/plain")


    @app.route("/litgpt_train_cmd")
    def litgpt_train_cmd():
        """
        This endpoint returns a simple HTML page that explains how to train LitGPT
        using the generated Anki JSON as a data source.
        """
        template = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>LitGPT Training Instructions</title>
            <meta charset="utf-8" />
            <style>
                body { font-family: sans-serif; margin: 2rem; }
                pre { background: #f5f5f5; padding: 0.5rem; }
                code { font-family: monospace; }
            </style>
        </head>
        <body>
            <h1>LitGPT Training Instructions</h1>
            <p>
                Below is a conceptual example of how you could train LitGPT on the Anki JSON
                generated by this server. You may need to adapt paths and commands depending
                on your LitGPT installation and version.
            </p>

            <h2>1. Download the Anki JSON</h2>
            <pre><code>curl -o anki_cards.json http://localhost:5000/anki_cards.json</code></pre>

            <h2>2. Convert JSON to a text training corpus</h2>
            <p>
                For example, create a simple text file where each line is a Q/A pair:
            </p>
            <pre><code>python -c "
    import json
    cards = json.load(open('anki_cards.json', 'r', encoding='utf-8'))
    with open('training_corpus.txt', 'w', encoding='utf-8') as f:
        for c in cards:
            f.write('Q: ' + c['front'].replace('\\n', ' ') + '\\n')
            f.write('A: ' + c['back'].replace('\\n', ' ') + '\\n\\n')
    "
            </code></pre>

            <h2>3. Example LitGPT training command</h2>
            <p>
                This is a generic example; adjust according to your installed LitGPT CLI:
            </p>
            <pre><code># Example: using a generic LitGPT trainer interface
    # (Replace &lt;model_name&gt; and flags with your actual setup)

    litgpt train \
        --model gpt2 \
        --data-path training_corpus.txt \
        --batch-size 4 \
        --max-steps 1000 \
        --learning-rate 3e-4 \
        --out-dir ./laegna_litgpt

    # After training, you might use:
    litgpt generate \
        --model-dir ./laegna_litgpt \
        --prompt "Q: Laegna AI Training Overview"
            </code></pre>

            <p>
                The key concept is: the Anki-derived training corpus contains structured
                question/answer examples enriched with chapter context. LitGPT can learn
                patterns of how questions map to answers and how context headings shape meaning.
            </p>

            <p><a href="/">Back to main page</a></p>
        </body>
        </html>
        """
        return Response(template, mimetype="text/html")


    if __name__ == "__main__":
        # Start Flask dev server
        # On Linux/Mac:
        #   python laegna_server.py
        # On Windows:
        #   python laegna_server.py
        app.run(host="0.0.0.0", port=5000, debug=True)
    =>```

---

## 4. How everything works (short recap)

- **Block reader (`[/BLOCKREAD/]`):**
  - Every code block and paragraph is stored with a `Q=<file>#blockN` / `A=<content>` pattern.
  - The model can learn consistent block‑level patterns and structure.

- **Chapter tree reader (`[/CHAPTREEREAD/]`):**
  - Tracks headings and builds a chapter tree, storing whole chapter content plus hierarchy.
  - Great for mapping table‑of‑contents‑style structures to full explanations.

- **Q/A tags (`[/Q]`, `[/A]`, `[/T]`):**
  - Parsed in a second pass.
  - Each Q/A -> one Anki card, with context from chapter tree.

- **Flask app:**
  - Serves rendered HTML, Anki JSON, TSV, and a LitGPT training instructions page.

- **LitGPT training:**
  - Use `anki_cards.json` → text corpus (`Q:` / `A:` lines) → `litgpt train ...`.
