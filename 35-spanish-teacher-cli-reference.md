# 35 - Spanish Teacher (CLI & Tools Reference)

While language learning doesn't have a traditional CLI, this reference covers the essential digital tools, syntax for flashcards, and prompt engineering for AI language tutors.

## 1. Anki SRS Syntax
Creating effective flashcards using Anki's bulk import formats.

### Basic CSV Import Format
```csv
Front,Back,Tags
"El perro","The dog","vocab animals"
"Yo ___ (hablar) español.","hablo","grammar present-tense"
```

### Cloze Deletion Format (Fill-in-the-blank)
```text
Text,Tags
"Ayer yo {{c1::fui}} al mercado.","grammar preterite"
"Si yo {{c1::tuviera}} dinero, {{c2::viajaría}} por el mundo.","grammar subjunctive conditional"
```

## 2. AI Tutor Prompts (ChatGPT/Claude)

### Conversation Practice
`Act as a native Spanish speaker from [Country]. Have a conversation with me at a [CEFR Level] level about [Topic]. Correct my mistakes gently after each turn and explain the grammar rule I violated.`

### Conjugation Drilling
`Generate a markdown table of the verb [Verb] in the [Tense] tense. Then, provide 5 fill-in-the-blank sentences using this verb in this tense.`

### Reading Comprehension
`Write a short story (300 words) in Spanish at a B1 level about a trip to Madrid. Include 5 comprehension questions at the end.`

## 3. Recommended Digital Tools
- **DeepL:** Superior to Google Translate for Spanish idioms and nuanced phrasing.
- **WordReference:** Excellent for seeing words in context across different regional dialects.
- **SpanishDict:** The definitive online dictionary with full conjugation tables and regional tags (e.g., [ES], [MX], [AR]).
- **Language Reactor:** Browser extension for Netflix/YouTube that provides dual subtitles and hover-translations.
