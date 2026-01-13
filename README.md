This repository will contain the "obvious basics" of AI training, and it's meant to inspire you: understand, what it means to bring your new knowledge and source creation to an AI, rather than new documents and document collections using well-known formats and structures.

Rather than collecting all kinds of information about RAG, fine-tuning and training here, and I have very small experience in training and fine-tuning, but I have been constantly resolving problems and tracking down the simple, essential idea - from implementation to practice.

My ready-made experience is such: it's hard to bring an innovative information to an AI without becoming somewhat experienced with some fine-tuning tools. You run into many problems of this lacking end-user support. A few web bots do this automatically on new information.

*Except this page, articles are written by CoPilot introducing my ideas one by one in clarified manner and background. I think it's a fairly standard information, but often the presentation is long: here, its only a few basic ideas and a single, simplest solution I could find; indeed you can grow further.*

# Fine-tuning patterns

To enter into the problems of lacking a conscious attention of an AI - training and fine doing -, having it running on routine patterns instead - RAG and other kinds of runtime memory -, you would run into goals of having your word understood.

You have the basic problems, identity patterns:
- **Minus**: [It does not understand your new vocabulary](Patterns/vocabulary.md).
- **Minus**: [It does not match your patterns properly](Patterns/rhythym.md).
- **Minus**: [its short-term contextual memory is weak](Patterns/compression.md).
- **Plus**: [It still extracts the basic meanings from your text, which are consistent with its patterns](Patterns/traditional.md).

Experiencing these patterns in *RAG* and local memory, you enter into the goals of implementing **training** and *fine-tuning* techniques and practices, which are not very automatic and clearly defined yet, for general community.

Read the texts to dive into the problems you are going to experience before you strongly choose towards techniques to get AI into conscious learning process.

# Fine-tuning compared to local context and lack of context

Structured data formats, such as database schemas and strict standards, are hard to manage as they run into exceptions and paradigms, so in this ancient word you become official, losing your soul and spirit. A.I. is a machine spirit: it combines metal, aether and space to give your spirit an automata.

You have to be agnostic, in your formats, for the ways you use your documentation:
- [Use-case-agnostic document collection](Agnostic/collections.md) - you need to work out the basic conventions on your data, and it's especially important that you yourself know them. Based on this, you can work further.
- [Use-case-agnostic document formats](Agnostic/agnosticformat.md) - UTF-8, Markdown and its dialects or more standard formats, and SpaCy for primitive translations and less contextual learning process which is still in the middle way to fine-tuning and training your own; each standard you create, indeed, brings another unawareness zone of your real case with all its examples, exceptions and intuitive cases based on your experiment and material, one-time needs.
- [Limitation of Realm of Forms: Q'n'A cards](Agnostic/QnA.md) - its very hard to understand, which teaching modes, especially if generalized, do not boil back to the extremely simple form of One Question with One Answer: inline calculators, context awareness and session with its chat history, etc., things which are clearly there in ChatGPT, still generally have to be expressed in this "rudimentary form", especially to be system-agnostic.
- [Some basic automation within its Realm of Bias and Confusion](Agnostic/dumbwork.md) - you have to automate Q&A extraction from your documents, and this creates dumb results. There could be some workaround in expressing them in proper patterns, such as warning about possible biases and confusions, and paralleling this with manual examples and even manual high-quality examples, which you might not be able to create a lot.
- [Automation of complex combinatorics](Agnostic/combinationsprogramming.md) - it's a serious advice, given by ChatGPT and CoPilot almost in any response, that you do not automate the Q&A, but create each card separately to reflect the real world rather than a theory, automation and dead routine of a machine. In creation of new theories and combinators, you run into many combinations you can only automate: there are no heavy bodies of expressions with all the dialects, different wordings, jokes, and special cases the users have found or simplifications in specific domains. One thing you can surely do is to label each Q&A pair for its limitations, and to give labels like "Quality" or "Seems fine" to the actual Q&A you manually create and compare with your whole quality flag.

# Finding a simple fine-tuning library - actual runnable tools

The fine-tuning library I am considering here and now is "LitGPT": this is a lightweight, more or less simple implementation of GPT model you can use for training or especially, to find the limitations and bugs in your training algorithm rather with local hardware and limited context window - "lit" might mean light, but it also conveys the message that you can get through within your limitations:
- [Learn the easiest to install and use, fine-tuning system](Training/LitGPT.md)
- [Understand some of the background](Background) - structure of Sessions, Identifiers and Tools the AI you teach can use, and how its still about the simple Q&A cards.
- [Multithread the parsers with IntelligentMistune before you start using SpaCy](IntelligentMistune) - you can use free-formed, general formats combined with specific tags, and extend mistune, python and flask to process your md files and repositories in somewhat dumb, but usable way if you also enter intelligent cards and let it find correlations between intelligent cards you can produce a few, with automated cards and summaries you can produce rather many, in multiple paradigm.
IntelligentMistune
- [An ultimate guide for programming with an AI: AI programming manual](https://github.com/tambetvali/LaegnaAIExperiments) - I think there is not much more to learn about AI, so the ultimate chapter in this guide is describing some technical details you might read or read loosely, depending on your understanding; but it tries to enable user to think on this topic even if they are far from programming: how to use an AI to program? More advanced users, indeed, can use programming ability, ability to express and express technically, and various levels of abilities to formulate them clearly, to get into decent results which could be sold; less intelligent, but now especially beginner users would resolve their particular problems on their own level, and they would as an AI to document the solution and to explain it in popular language, or describe the use cases, risks and possibilities: such users can still resolve a lot, even if they do not master 7 key traits of roman aristocrat, starting from "Speaking Ability" (formalize or express the task), Courage (do something useful or at last, protect from hackers by adding password and not exposing your personal and companie's protected data), etc.
- Afterthought: [Understand your common abilities with IntelligentGUI before you go mad](IntelligentGUI) - I started to ponder, whether I made clear its for readers, users with all background, not a very technical guide; to make clear one understands this scope which is usable for advanced, intermediate and beginner users, as well as the ones who are interested only in having the data source rather than inventing it or spending time to make it shine: so here is a definite guide which shows the steps if you *do not* have even command line ability; but you can listen AI in *very basic* command line use: my sister, for example, is not a master of command line and never uses it, but I remember in 386 to run games and programs, everybody could type those lines from paper to its only initial interface, so listening to AI is not exactly command line professional, but some sensible work.
