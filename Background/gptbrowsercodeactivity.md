# GPT Context Window, Session Memory, and Tool-Capable Assistants  
### (Anki Card, HTML View, Anki Internal Format, Python Reader/Writer)

This document explains:

- How a **context window** works  
- How **session memory** is inserted  
- Which **types of GPT-style assistants** support which capabilities  
- How a program might process a **JSON representation** of a Q&A card  
- How to represent this in **Anki fields**, **templates**, and **HTML**  
- How Python pseudocode might simulate the assistant’s behavior  

---

# 1. Context Window & Session Memory (Short Explanation)

A **context window** is the text the model can see at once.  
It has a fixed size (e.g., 8k, 32k, 128k tokens).  
Anything outside it is forgotten unless reinserted.

**Session memory** is external.  
The system stores facts, summaries, or previous messages and reinserts them into the prompt.

No special syntax is required; tags like `[System]`, `[Memory]`, `[User]` are conventions.

---

# 2. Which Types of GPT-Style Assistants Support Which Capabilities?

Below is a general classification of assistant types and what they typically support.

## A. Web-enabled assistants (e.g., Copilot, ChatGPT with browsing)
**Can do:**
- Web search  
- Read first page of a web resource  
- Use built-in calculator  
- Use project files (if integrated into an IDE)  

**Cannot do:**
- Modify arbitrary local files without explicit user permission  
- Access private resources unless provided  

---

## B. Local offline LLMs (no tools)
**Can do:**
- Pure text reasoning  
- Math only through internal reasoning (no calculator)  

**Cannot do:**
- Web search  
- Read web pages  
- Access files  
- Use external tools  

---

## C. Code assistants (IDE-integrated)
**Can do:**
- Read project files  
- Apply changes to files  
- Use built-in static analysis  
- Sometimes use a calculator  

**Cannot do:**
- Web search (unless connected to a tool)  

---

## D. Math-enhanced assistants
**Can do:**
- Use a built-in calculator  
- Perform symbolic math  

**Cannot do:**
- Web search  
- File operations  

---

# 3. Example Anki Note Fields

```
Topic: GPT Context Window & Tool Capabilities

Background:
Different types of GPT-style assistants support different capabilities.
Some can search the web, some can use a calculator, some can read files.

UserMessage:
Which types of GPT assistants can perform web search, use a calculator, read a web page, or modify project files?

SessionMemory:
- User name: Alex
- Interest: AI internals
- Previous topic: context windows

ContextWindowExample:
[System] You are a helpful assistant.
[Memory] User name: Alex.
[History] User: Explain context windows.
[History] Assistant: A context window is...
[User] Which types of GPT assistants can perform web search, use a calculator, read a web page, or modify project files?

AnswerExplanation:
Web-enabled assistants can search and read pages.
Code assistants can modify project files.
Math-enhanced assistants can use calculators.
Offline LLMs cannot use external tools.
```

---

# 4. Anki Card Templates

## Front (Q → A)

```html
<div class="topic">{{Topic}}</div>

<h2>Q:</h2>
<div class="question">{{UserMessage}}</div>

<h3>Session Memory:</h3>
<div class="memory">{{SessionMemory}}</div>

<h3>Context Window Example:</h3>
<div class="context">{{ContextWindowExample}}</div>
```

## Back (A)

```html
<h2>A:</h2>
<div class="answer">{{AnswerExplanation}}</div>

<hr>

<h3>Background:</h3>
<div class="background">{{Background}}</div>
```

---

# 5. Reverse Card (A → Q)

```html
<h2>A (Given):</h2>
<div class="answer">{{AnswerExplanation}}</div>

<hr>

<h2>Original Question:</h2>
<div class="question">{{UserMessage}}</div>
```

---

# 6. CSS for Anki or Browser

```css
.topic {
  font-size: 1.2em;
  font-weight: bold;
  color: #3366cc;
}

.question, .answer, .memory, .context, .background {
  margin: 10px 0;
  padding: 8px;
  border-left: 3px solid #ccc;
  background-color: #f9f9f9;
}
```

---

# 7. Browser HTML Rendering

```html
<div class="card">
  <div class="topic">GPT Context Window & Tool Capabilities</div>

  <div class="label">Q:</div>
  <div class="box">
Which types of GPT assistants can perform web search, use a calculator, read a web page, or modify project files?
  </div>

  <div class="label">Session Memory:</div>
  <div class="box">
- User name: Alex
- Interest: AI internals
- Previous topic: context windows
  </div>

  <div class="label">Context Window Example:</div>
  <div class="box">
[System] You are a helpful assistant.
[Memory] User name: Alex.
[History] User: Explain context windows.
[History] Assistant: A context window is...
[User] Which types of GPT assistants can perform web search, use a calculator, read a web page, or modify project files?
  </div>
</div>
```

---

# 8. Internal Anki JSON-like Representation

```json
{
  "note_type": "GPTContextWindowTools",
  "fields": {
    "Topic": "GPT Context Window & Tool Capabilities",
    "Background": "Different types of GPT-style assistants support different capabilities.",
    "UserMessage": "Which types of GPT assistants can perform web search, use a calculator, read a web page, or modify project files?",
    "SessionMemory": "- User name: Alex\n- Interest: AI internals\n- Previous topic: context windows",
    "ContextWindowExample": "[System] You are a helpful assistant.\n[Memory] User name: Alex.\n[History] User: Explain context windows.\n[History] Assistant: A context window is...\n[User] Which types of GPT assistants can perform web search, use a calculator, read a web page, or modify project files?",
    "AnswerExplanation": "Web-enabled assistants can search and read pages. Code assistants can modify project files. Math-enhanced assistants can use calculators. Offline LLMs cannot use external tools."
  }
}
```

---

# 9. Python Pseudocode: Reader, Writer, JSON Converter, and Assistant Simulation

```python
# -----------------------------------------
# 1. Load the Anki-like JSON card
# -----------------------------------------

import json

with open("card.json", "r") as f:
    card = json.load(f)

fields = card["fields"]

# -----------------------------------------
# 2. Render templates (simplified)
# -----------------------------------------

from string import Template

front_template = Template("""
<div class="topic">${Topic}</div>
<h2>Q:</h2>
<div class="question">${UserMessage}</div>
""")

front_html = front_template.substitute(fields)

# -----------------------------------------
# 3. Simulate different GPT-style assistants
# -----------------------------------------

def simulate_assistant(assistant_type, card_fields):
    """
    assistant_type: "web", "offline", "math", "code"
    """

    if assistant_type == "web":
        # Web-enabled assistant
        # Can perform web search, read pages, use calculator
        return {
            "capabilities": ["web_search", "read_web_page", "calculator"],
            "response": f"Using web tools, I can answer: {card_fields['AnswerExplanation']}"
        }

    if assistant_type == "offline":
        # Offline LLM
        return {
            "capabilities": [],
            "response": "I cannot use external tools, but I can reason about the question."
        }

    if assistant_type == "math":
        return {
            "capabilities": ["calculator"],
            "response": "I can compute things but cannot search the web."
        }

    if assistant_type == "code":
        return {
            "capabilities": ["read_project_files", "apply_file_changes"],
            "response": "I can modify project files based on the question."
        }

# -----------------------------------------
# 4. Convert card to JSON for the assistant
# -----------------------------------------

assistant_input = {
    "question": fields["UserMessage"],
    "memory": fields["SessionMemory"],
    "context": fields["ContextWindowExample"]
}

assistant_json = json.dumps(assistant_input, indent=2)

# -----------------------------------------
# 5. Run through mock assistant
# -----------------------------------------

result = simulate_assistant("web", fields)
print(result)
```

---

# 10. Summary

This document shows:

- How **context windows** and **session memory** work  
- How different **types of GPT-style assistants** support different capabilities  
- How to represent this in **Anki fields**, **HTML**, and **JSON**  
- How Python pseudocode might simulate the assistant’s behavior  

Everything is structured so you can study, modify, or extend it.
