# Context Window, Session Memory, and Anki Integration

This document explains:

1. How a **context window** and **session memory** work.  
2. How to represent them in **Anki Q&A cards** (note fields, card templates, CSS).  
3. How a **browser-like HTML Q&A view** might look.  
4. A conceptual **Anki card format** covering these details.  
5. **Python code** to generate an Anki-like card, read it, parse to JSON, and send to a mock AI interface.

---

## 1. Short Explanation: Context Window & Session Memory

- **Context window**:  
  The text the model can see at once. It has a fixed size (N tokens). Anything outside it is invisible unless reinserted.

- **Session memory**:  
  Stored externally (e.g., database, file). Before each response, relevant session memory is **inserted into the context window** as plain text (often with simple tags or labels).

No special syntax is required by the model itself; tags like `[System]`, `[User]`, `[Memory]` are conventions to structure the prompt.

---

## 2. Example Anki Note Fields

We define a note type that captures how ChatGPT / Copilot-like behavior works:

**Fields:**
- `Topic`
- `Background`
- `UserMessage`
- `SessionMemory`
- `ContextWindowExample`
- `AnswerExplanation`

Example data you might put into one Anki note:

```
Topic: Context Windows & Session Memory

Background:
Large language models do not store long-term memory internally.
They rely on the context window and external session memory.

UserMessage:
How does the context window work, and how can session memory be represented?

SessionMemory:
- User name: Alex
- Preference: concise answers
- Previous question: "Explain transformers simply"
- Summary: User is learning about LLM internals.

ContextWindowExample:
[System] You are a helpful assistant.
[Memory] User name: Alex. Prefers concise answers.
[History] User: Explain transformers simply.
[History] Assistant: A transformer is...
[User] How does the context window work?

AnswerExplanation:
The context window is the text the model sees at once.
Session memory is external and must be reinserted into the prompt.
```

---

## 3. Anki Card Templates (Q → A and A → Q)

### 3.1. Card 1: Question → Answer (Front / Back)

**Front Template (Q):**

```html
<div class="topic">{{Topic}}</div>

<h2>Q:</h2>
<div class="question">{{UserMessage}}</div>

<h3>Session Memory:</h3>
<div class="memory">{{SessionMemory}}</div>

<h3>Context Window Example:</h3>
<div class="context">{{ContextWindowExample}}</div>
```

**Back Template (A):**

```html
<h2>A:</h2>
<div class="answer">{{AnswerExplanation}}</div>

<hr>

<h3>Full Background:</h3>
<div class="background">{{Background}}</div>
```

---

### 3.2. Card 2: Answer → Question (Reverse)

**Front Template (A → Q):**

```html
<h2>A (Given):</h2>
<div class="answer">{{AnswerExplanation}}</div>

<hr>

<h2>Original Question:</h2>
<div class="question">{{UserMessage}}</div>
```

**Back Template (Q):**

```html
<div class="topic">{{Topic}}</div>

<h2>Q (Recall):</h2>
<div class="question">{{UserMessage}}</div>

<h3>Session Memory:</h3>
<div class="memory">{{SessionMemory}}</div>
```

---

### 3.3. CSS Styling for Anki Card

```css
.topic {
  font-size: 1.2em;
  font-weight: bold;
  color: #3366cc;
  margin-bottom: 10px;
}

.question, .answer, .memory, .context, .background {
  margin: 10px 0;
  padding: 8px;
  border-left: 3px solid #ccc;
  background-color: #f9f9f9;
}

h2, h3 {
  margin: 8px 0 4px 0;
  font-family: sans-serif;
}

body {
  font-family: sans-serif;
  font-size: 14px;
}
```

---

## 4. Browser-style Q&A View (HTML + Design)

Imagine a simple web page previewing the card similarly to Anki’s browser:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Context Window & Session Memory Q&A</title>
  <style>
    body {
      font-family: sans-serif;
      margin: 20px;
    }
    .card {
      border: 1px solid #ddd;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 4px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.05);
    }
    .topic {
      font-size: 1.2em;
      font-weight: bold;
      color: #3366cc;
      margin-bottom: 10px;
    }
    .label {
      font-weight: bold;
      color: #333;
      margin-top: 10px;
    }
    .box {
      margin: 5px 0 10px 0;
      padding: 8px;
      border-left: 3px solid #ccc;
      background-color: #f9f9f9;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>

<div class="card">
  <div class="topic">Context Windows & Session Memory</div>

  <div class="label">Q:</div>
  <div class="box">
How does the context window work, and how can session memory be represented?
  </div>

  <div class="label">Session Memory:</div>
  <div class="box">
- User name: Alex
- Preference: concise answers
- Previous question: "Explain transformers simply"
- Summary: User is learning about LLM internals.
  </div>

  <div class="label">Context Window Example:</div>
  <div class="box">
[System] You are a helpful assistant.
[Memory] User name: Alex. Prefers concise answers.
[History] User: Explain transformers simply.
[History] Assistant: A transformer is...
[User] How does the context window work?
  </div>
</div>

<div class="card">
  <div class="topic">Context Windows & Session Memory</div>

  <div class="label">A:</div>
  <div class="box">
The context window is the text the model sees at once.
Session memory is external and must be reinserted into the prompt.
  </div>

  <div class="label">Background:</div>
  <div class="box">
Large language models do not store long-term memory internally.
They rely on the context window and external session memory.
  </div>
</div>

</body>
</html>
```

---

## 5. Conceptual Anki Card Format (JSON-like Representation)

Here is how a single note could be represented in a JSON-like format before becoming Anki cards:

```json
{
  "note_type": "ContextWindowSessionMemory",
  "fields": {
    "Topic": "Context Windows & Session Memory",
    "Background": "Large language models do not store long-term memory internally.\nThey rely on the context window and external session memory.",
    "UserMessage": "How does the context window work, and how can session memory be represented?",
    "SessionMemory": "- User name: Alex\n- Preference: concise answers\n- Previous question: \"Explain transformers simply\"\n- Summary: User is learning about LLM internals.",
    "ContextWindowExample": "[System] You are a helpful assistant.\n[Memory] User name: Alex. Prefers concise answers.\n[History] User: Explain transformers simply.\n[History] Assistant: A transformer is...\n[User] How does the context window work?",
    "AnswerExplanation": "The context window is the text the model sees at once.\nSession memory is external and must be reinserted into the prompt."
  },
  "cards": [
    {
      "name": "Forward",
      "front_template": "<div class=\"topic\">{{Topic}}</div>\n\n<h2>Q:</h2>\n<div class=\"question\">{{UserMessage}}</div>\n\n<h3>Session Memory:</h3>\n<div class=\"memory\">{{SessionMemory}}</div>\n\n<h3>Context Window Example:</h3>\n<div class=\"context\">{{ContextWindowExample}}</div>",
      "back_template": "<h2>A:</h2>\n<div class=\"answer\">{{AnswerExplanation}}</div>\n\n<hr>\n\n<h3>Full Background:</h3>\n<div class=\"background\">{{Background}}</div>"
    },
    {
      "name": "Reverse",
      "front_template": "<h2>A (Given):</h2>\n<div class=\"answer\">{{AnswerExplanation}}</div>\n\n<hr>\n\n<h2>Original Question:</h2>\n<div class=\"question\">{{UserMessage}}</div>",
      "back_template": "<div class=\"topic\">{{Topic}}</div>\n\n<h2>Q (Recall):</h2>\n<div class=\"question\">{{UserMessage}}</div>\n\n<h3>Session Memory:</h3>\n<div class=\"memory\">{{SessionMemory}}</div>"
    }
  ]
}
```

---

## 6. Python Code: Generate Anki-like Card, Read It, Parse to JSON, Send to Mock AI

This is a **conceptual example** showing:

1. How you might define a note in Python.
2. How to render templates.
3. How to serialize to JSON.
4. How to send it to a mock AI interface.

```python
import json
from string import Template

# 1. Define the raw note data (like an Anki note)
note = {
    "note_type": "ContextWindowSessionMemory",
    "fields": {
        "Topic": "Context Windows & Session Memory",
        "Background": (
            "Large language models do not store long-term memory internally.\n"
            "They rely on the context window and external session memory."
        ),
        "UserMessage": "How does the context window work, and how can session memory be represented?",
        "SessionMemory": (
            "- User name: Alex\n"
            "- Preference: concise answers\n"
            "- Previous question: \"Explain transformers simply\"\n"
            "- Summary: User is learning about LLM internals."
        ),
        "ContextWindowExample": (
            "[System] You are a helpful assistant.\n"
            "[Memory] User name: Alex. Prefers concise answers.\n"
            "[History] User: Explain transformers simply.\n"
            "[History] Assistant: A transformer is...\n"
            "[User] How does the context window work?"
        ),
        "AnswerExplanation": (
            "The context window is the text the model sees at once.\n"
            "Session memory is external and must be reinserted into the prompt."
        )
    }
}

# 2. Define simple templates using Python's Template (for demonstration)

front_template_forward = Template(
    """<div class="topic">${Topic}</div>

<h2>Q:</h2>
<div class="question">${UserMessage}</div>

<h3>Session Memory:</h3>
<div class="memory">${SessionMemory}</div>

<h3>Context Window Example:</h3>
<div class="context">${ContextWindowExample}</div>"""
)

back_template_forward = Template(
    """<h2>A:</h2>
<div class="answer">${AnswerExplanation}</div>

<hr>

<h3>Full Background:</h3>
<div class="background">${Background}</div>"""
)

# 3. Render the card as HTML (simulating Anki's template rendering)

fields = note["fields"]
front_html = front_template_forward.substitute(fields)
back_html = back_template_forward.substitute(fields)

# 4. Create a serializable card object

card = {
    "note_type": note["note_type"],
    "front": front_html,
    "back": back_html,
    "fields": fields
}

# 5. Serialize card to JSON (e.g., for storage or sending via API)
card_json = json.dumps(card, indent=2)
print("Serialized Card JSON:")
print(card_json)

# 6. Mock AI interface that receives the card and returns a "response"

def mock_ai_interface(card_json_str):
    data = json.loads(card_json_str)
    # Pretend the AI reads the context window example and explanation
    # and produces a summary message about how context and session memory work.
    topic = data["fields"]["Topic"]
    user_msg = data["fields"]["UserMessage"]
    answer = data["fields"]["AnswerExplanation"]

    response = {
        "summary": (
            f"This card teaches the concept '{topic}'. "
            f"It starts from the user question: '{user_msg}'. "
            f"The core idea is: {answer.splitlines()[0]}"
        ),
        "original_card": data
    }
    return response

# 7. Send the card JSON to the mock AI and print the response

ai_response = mock_ai_interface(card_json)
print("\nMock AI Response:")
print(json.dumps(ai_response, indent=2))
```

---

## 7. How This Reflects Real ChatGPT / Copilot Behavior

- The **ContextWindowExample** field mimics the **prompt** the AI actually sees:
  - system message
  - memory
  - history
  - current user message

- The **SessionMemory** field represents **external memory**:
  - user profile
  - previous questions
  - summaries

- The **AnswerExplanation** field is like the model output.

- The Python code simulates:
  - storing the card,
  - rendering the view,
  - serializing it,
  - sending it to a mock AI interface that “understands” the context.

This gives you a compact, realistic model of how context windows + session memory + Q&A cards can all be reasoned about in one coherent structure.
