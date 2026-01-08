# LitGPT: Practical Overview and How to Feed It Training Cards

LitGPT is designed as an **installable product**, not a fragile dependency stack. Unlike Hugging Face Transformers — where version conflicts are common — LitGPT installs as a **self‑contained environment** with its own compatible dependencies. This makes it ideal for users who want a “black‑box” installation that *just works*.

LitGPT provides:

- A **command‑line interface (CLI)** for downloading models, fine‑tuning, pretraining, inference, and evaluation.  
- A **Python API** for dynamic or programmatic workflows.  
- Support for **JSON datasets**, **YAML recipes**, and **custom data loaders**.  
- Ready‑made tutorials for **fine‑tuning**, **pretraining**, **LoRA**, **QLoRA**, and more.

Below is a practical guide to using LitGPT for Q&A‑style training, including command‑line examples, server‑based workflows, and card formats.

---

# 1. Installing LitGPT

LitGPT installs cleanly:

\`\`\`bash
pip install litgpt[all]
\`\`\`

This installs:

- LitGPT core  
- Tokenizers  
- Flash‑attention  
- Training utilities  
- CLI tools  

No dependency juggling required.

---

# 2. Downloading a Model (CLI)

LitGPT can download models directly from Hugging Face:

\`\`\`bash
litgpt download microsoft/phi-2
\`\`\`

You can list all available models:

\`\`\`bash
litgpt download list
\`\`\`

---

# 3. Feeding LitGPT Training Cards (CLI)

LitGPT fine‑tuning expects **JSON datasets** in Alpaca‑style or instruction‑style format.

Example command:

\`\`\`bash
litgpt finetune microsoft/phi-2 \
  --data JSON \
  --data.json_path my_cards.json \
  --data.val_split_fraction 0.1
\`\`\`

### Format of the JSON file

LitGPT supports Alpaca‑style entries:

\`\`\`json
[
  {
    "instruction": "Question text here",
    "input": "",
    "output": "Answer text here"
  }
]
\`\`\`

Or simple Q&A:

\`\`\`json
[
  {
    "question": "What is X?",
    "answer": "X is ..."
  }
]
\`\`\`

LitGPT automatically tokenizes and formats these.

---

# 4. Using a Custom Server to Serve Cards

You can serve your Q&A cards from:

- A Flask server  
- A static JSON endpoint  
- A local file server  
- An Anki export converted to JSON  

LitGPT does **not** require a database — any JSON file or URL is acceptable.

### Example: Serving cards via Flask

\`\`\`python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/cards")
def cards():
    return jsonify([
        {"instruction": "What is 2+2?", "output": "4"},
        {"instruction": "Define entropy.", "output": "Entropy is ..."}
    ])

app.run()
\`\`\`

Then train with:

\`\`\`bash
litgpt finetune microsoft/phi-2 \
  --data JSON \
  --data.json_path http://localhost:5000/cards
\`\`\`

---

# 5. Dynamic Execution: Feeding Cards from Python

LitGPT has a Python API for dynamic dataset creation.

Example:

\`\`\`python
from litgpt import Trainer, Config

cards = [
    {"instruction": "Explain gravity.", "output": "Gravity is ..."},
    {"instruction": "What is a neuron?", "output": "A neuron is ..."}
]

cfg = Config(model="microsoft/phi-2", data=cards)
trainer = Trainer(cfg)
trainer.finetune()
\`\`\`

You can generate cards on the fly, load them from Anki, or synthesize them programmatically.

---

# 6. Supported Card Formats

LitGPT supports several dataset formats:

| Format | Supported? | Notes |
|--------|------------|-------|
| **Q&A** | ✔️ Yes | Simplest format; works everywhere |
| **Instruction + Input + Output** | ✔️ Yes | Alpaca‑style; widely used |
| **Documentation → Q&A** | ✔️ Yes | Must be converted to JSON |
| **ChatML / Multi‑turn** | ✔️ Partial | Requires custom data loader |
| **Anki Q&A** | ❌ Not directly | Must be exported → JSON |

### Do all formats boil down to Q&A?

**Yes.**  
Even instruction‑style datasets are fundamentally Q&A:

- *Instruction* = Question  
- *Output* = Answer  

LitGPT treats them as prompt → completion pairs.

### Are Anki cards always supported?

Not directly.

But Anki exports to:

- CSV  
- JSON (via add‑ons)  
- TSV  

These can be trivially converted to LitGPT JSON format.

Thus: **Anki → JSON → LitGPT** is the standard workflow.

---

# 7. Recommended Workflow for Your Use‑Case‑Agnostic System

### 1. Write everything in UTF‑8 Markdown  
Your universal source format.

### 2. Convert Markdown → Q&A JSON  
Using:

- SpaCy  
- DatSu  
- Python scripts  
- Custom parsers  

### 3. Export to Anki for human review  
Anki ensures clarity and correctness.

### 4. Export Anki → JSON for LitGPT  
Final training dataset.

### 5. Train using CLI or Python API  
Depending on your automation needs.

---

# 8. Command‑Line Examples (Escaped Code Blocks)

### Download a model

\`\`\`bash
litgpt download microsoft/phi-2
\`\`\`

### Fine‑tune on your Q&A cards

\`\`\`bash
litgpt finetune microsoft/phi-2 \
  --data JSON \
  --data.json_path my_cards.json
\`\`\`

### Run inference

\`\`\`bash
litgpt inference microsoft/phi-2 \
  --prompt "Explain entropy."
\`\`\`

### Evaluate model

\`\`\`bash
litgpt evaluate microsoft/phi-2 \
  --data JSON \
  --data.json_path test_cards.json
\`\`\`

---

# 9. Resources to Start

- LitGPT Home: https://lightning.ai/docs/litgpt  
- CLI Interface: https://deepwiki.com/Lightning-AI/litgpt/7.4-cli-interface  
- Tutorials (finetuning, pretraining, inference):  
  https://github.com/Lightning-AI/litgpt/tree/main/tutorials  
- PyPI package: https://pypi.org/project/litgpt

---

# Final Answers to Your Questions

### 1. Command line to feed LitGPT training cards?  
Use:

\`\`\`bash
litgpt finetune model-name --data JSON --data.json_path file.json
\`\`\`

### 2. Custom server to serve cards?  
Yes — any JSON endpoint works.

### 3. Dynamic execution via Python or CLI?  
Yes — Python API supports dynamic datasets.

### 4. Which card formats are supported?  
- Q&A  
- Instruction → Output  
- Instruction + Input → Output  
- Documentation converted to Q&A  

### 5. Are they all boiled down to Q&A?  
**Yes.**  
All formats reduce to “prompt → completion.”

### 6. Are Anki Q&A formats always supported?  
Not directly — but **Anki → JSON → LitGPT** works perfectly.
