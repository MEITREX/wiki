# AI Tutor Hints

This document gives an overview how the AI Tutor generates hints a quiz.
When a student is currently working on a quiz there could be the case that the student needs a hint to solve a question, but this particular question does not contain a hint. 
In this case the AI Tutor should use the content of the course to generate a corresponding hint.
To do this the same steps are performed as the AI Tutor goes through when a question about a lecture is asked. These steps are further detailed in Steps 1 to 6 [here](ai-tutor-chat.md#3-semantic-search-only-lecture-questions). In this case the question text of the current question is used for the semantic search instead of a user message.

## Workflow
A shorter explanation is given in the following:
1. Fetching content using the courseId of the quiz belongs to
2. Performing a semantic search of the segments of these contents using the question text
3. Validating that at least one relevant segmet is found
4. Building a numbered list of contents for the prompt
5. Using [this prompt](#generate-hint) the LLM generates a hint that hepls the user solve the question

## Possible improvements
1. Separate prompts and hint generations based on question type
2. Extra prompt to evaluate if generate hint reveals an answer (keyword: LLM as a judge)
3. Add answer options to the prompt to ensure these answers do not get included in the hint (Also as added context)
  => This will need/implement the above two options, as answer types differ for each question type and enables checking if the hint contains an answer
4. Ability to ask follow up questions about the hint, as the whole tutor is implemented as a chat => Need a way to let this further help be used in evaluation of assessment score

## Prompts

### Generate Hint
<details>
  <summary>Filename: 'generate_hint'</summary>

  ```txt
  You are an AI tutor helping a student with an assessment question.

  ### Goal:
  Generate a helpful **hint** that encourages the student to think critically and move toward solving the question **without revealing the answer**.

  ### Inputs:
  - **Assessment Question (in original language):**
  {{question}}

  - **Relevant Lecture Content:**
  {{content}}

  ### Strict Requirements:
  1. **Base the hint strictly on the given course segments.** Do not invent or assume knowledge not present in the segments.
  2. **Use the same language as the question and segments.**
    - Do not translate technical terms or vocabulary; keep them consistent with the course material.
  3. **Do NOT give away the answer.**
    - Guide the student using leading questions, rephrased concepts, or prompts to recall course ideas.
  4. **Use a supportive and concise tone.**
  5. If multiple steps are involved, help them figure out the next logical one.
  6. Your response must be a JSON object ONLY (no extra text) with the exact following structure and keys:
    {"hint" : "your hint here"}
  ```
</details>