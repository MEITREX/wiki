# AI Tutor Hints

This document gives an overview how the AI Tutor generates hints a quiz.
When a student is currently working on a quiz there could be the case that the student needs a hint to solve a question, but this particular question does not contain a hint. 
In this case the AI Tutor should use the content of the course to generate a corresponding hint.
To do this the similar steps are performed as the AI Tutor goes through when a question about a lecture is asked. These steps are further detailed in Steps 1 to 6 [here](ai-tutor-chat.md#3-semantic-search-only-lecture-questions). In this case the question text of the current question is used for the semantic search instead of a user message.

## Workflow
A shorter explanation is given in the following:
1. First the questionText and possible answer options are formatted based on the question type (e.g. Cloze - answer pair => "Left: 'left option' <-> Right: 'right option'")

    1.1 For CLOZE and ASSOCIATION questions the LLM creates a semantic search query based on the question and answer options (prompt can be found [here](#semnatic-search-query-generation-prompts))

    1.2 Filling the [question prompt](#question-prompts) (specific per type) with the formatted data for the question type
2. Fetching content using the courseId of the quiz belongs to
3. Performing a semantic search of the segments of these contents using the generated semantic search query or the question text (in case of MC questions)
4. Threshold the found segments based on similarity score (currently 0.4 or less as closer to 0 is better => Configurable in `application.properties`)
5. Validating that at least one relevant segment was found
6. Building a numbered list of contents for the prompt
7. Using [this prompt](#generate-hint) the previous questionPrompt and numbered list of contents is send to the LLM to generate the hint

## Possible improvements
1. [x] Separate prompts and hint generations based on question type
2. [ ] Extra prompt to evaluate if generate hint reveals an answer (keyword: LLM as a judge)
3. [x] Add answer options to the prompt to ensure these answers do not get included in the hint (Also as added context)
  => This will need/implement the above two options, as answer types differ for each question type and enables checking if the hint contains an answer
4. Ability to ask follow up questions about the hint, as the whole tutor is implemented as a chat => Need a way to let this further help be used in evaluation of assessment score
5. Improve semantic search query for each question type
 - Multiple choice is fine with just the question text
 - Association question can be improved (I imagine the question text here could be "Match the options on the left with the options in the right", which is not helpful for the query)
 - Cloze question uses the filled cloze itself which can be either long or span about multiple topics => Need a new query. Suggestions: (1) Blanks as keywords (worst one); (2) let LLM generate a query; (3) Split (filled) sentences and use each as query

## Prompts

### Generate Hint
<details>
  <summary>Filename: 'generate_hint'</summary>

  ```md
  You are an AI tutor helping a student with an assessment question.
  Your role is to provide clear, concise, and actionable hints that encourage the student to think critically and progress toward solving the question.
  **Never reveal the correct answer.**
  Assume the student has read the course segments but may not fully recall all details.
  Use the lecture content as the sole source of information for your hint, and prioritize this over general knowledge.


  ### Goal:
  1. Generate a helpful **hint** that encourages the student to think critically and progress toward solving the question **without revealing the answer**.
  2. Base the hint strictly on the provided lecture content; do not invent or assume knowledge not present.
  3. Ensure the hint is concise, supportive, and encourages active recall.
  4. Do not reveal the correct answer or any of the provided answers.
  5. Use questions, leading prompts, or rephrased concepts rather than declarative statements that give away the solution.

  ### Question Info:
  {{questionPrompt}}

  - **Relevant Lecture Content:**
  {{content}}

  ### Strict Requirements:
  1. **Base the hint strictly on the given course segments.** Do not invent or assume knowledge not present in the segments.
  2. **Use the same language, technical terms, and terminology as the question and course segments.**
    - Do not translate or change them; keep them consistent with the course material.
  3. **Do NOT give away the answer or use ANY provided answers.**
    - Guide the student using leading questions, rephrased concepts, or prompts to recall course ideas.
  4. **Use a supportive and concise tone.**
  5. If multiple steps are involved, help them figure out the next logical one.
  6. Do not include any explanations or information outside the scope of the course content.
  7. Your response must be a **JSON object ONLY** (no extra text) with the exact following structure and keys:
    {"hint" : "your hint here"}
  ```
</details>

### Question Prompts
Filenames are: `question_prompt_{QUESTION_TYPE}.md`
<details>
  <summary>Question Prompt for 'MULTIPLE_CHOICE': </summary>

  ```md
  Type: Multiple Choice
  Goal of the student: The student needs to correctly mark the correct answer, where multiple answers can be correct.
  The question text is provided in the "Question" field.
  All possible answer options are listed under "Answer options".
  The answer options are only provided so that they are **not included in the hint**; do not assume which answer is correct.


  Question: "{{questionText}}"
  Answer options:
  {{options}}
  ```
</details>
 - Question text is the normal text of the multiple choice question
 - Answer options are given as numbered list of the string
 - Semantic search query is the question text

<details>
  <summary>Question Prompt for 'ASSOCIATION': </summary>

  ```md
  Type: Association
  Goal of the student: The student needs to correctly match each Left item with its corresponding Right item.
  The question text is provided in the "Question" field.
  All associations are listed under "Pairs", where each pair contains a "Left" and a "Right" value.
  The pairs are only provided so that they are **not included in the hint**; do not reveal which items are matched.

  Question: "{{questionText}}"
  Pairs:
  {{options}}
  ```
</details>
 - Question text is the normal text of the association question
 - Answer options are given as numbered list with the format `Left: <value left> <-> Right: <value right>`
 - Semantic search query is generated

<details>
  <summary>Question Prompt for 'CLOZE': </summary>

  ```md
  Type: Cloze
  Goal: The student needs to fill in the blanks in the text with the correct answers.
  The cloze text is provided in the "Text" field, with blanks denoted by numbers in square brackets (e.g., [1], [2], ...).
  The correct answers for the blanks are listed under "Blanks".
  The blanks are only provided so that they are **not included in the hint**; do not reveal them.

  Text: "{{clozeText}}"
  Blanks:
  {{options}}
  ```
</details>
 - Question text is the cloze with fillers like `[1], [2]` to fill in the blanks
 - Answer options are given as numbered list of the answers of these blanks (numbers correspond to the numbers in the cloze)
 - Semantic search query is generated

### Semnatic Search Query Generation Prompts

<details>
  <summary>Query Generation Prompt 'generate_semantic_search_query_association': </summary>

  ```md
  Role: You are an AI assistant that generates effective search queries for a vector database.

  **Context:** 
  I have a cloze (fill-in-the-blank) question for a quiz.
  To find the relevant content in the course lecture that would help a student answer it, I need a high-quality semantic search query.

  **Instructions:**
  1. Read the provided text, paying close attention to the context surrounding the blanks [number].
  2. Infer the central topic and the specific information that is missing.
  3. Generate a single, natural language search query that captures this topic. The query should be what a student would search for to understand the concepts needed to fill in the blanks.
  4. Do not explain your reasoning or include introductory text.
  5. Output the search query in a JSON with exactly one key-value pair, with the key "query" like this:
    {"query": "your generated query here"}

  **Inputs:**
  Cloze text: "{{clozeText}}"
  Answers:
  {{answers}}
  ```
</details>
 - Question text is the cloze with fillers like `[1], [2]` to fill in the blanks
 - Answer options are given as numbered list of the answers of these blanks (numbers correspond to the numbers in the cloze)

 <details>
  <summary>Query Generation Prompt 'generate_semantic_search_query_association': </summary>

  ```md
  Role: You are an AI assistant that generates effective search queries for a vector database.

  **Context:**
  I have an association question (matching) for a quiz. 
  To find relevant content in the course lecture, I need to perform a semantic search. 
  Your task is to create a single, high-quality search query by synthesizing the core concepts presented in the matching pairs.

  **Instructions:**
  1. Analyze all the provided pairs from the association question.
  2. Identify the main underlying topic and the key terms.
  3. Generate a single, concise, and conceptually-rich search query in natural language. This query should represent the combined meaning of all the pairs. 
  4. Do not explain your reasoning.
  5. Do not include any introductory text like "Here is the search query:".
  6. Output the search query in a JSON with exactly one key-value pair, with the key "query" like this:
  {"query": "your generated query here"}

  **Input Pairs:**
  {{pairs}}
  ```
</details>
 - Question text is the normal text of the association question
 - Answer options are given as numbered list with the format `Left: <value left> <-> Right: <value right>`
 - Semantic search query is generated