# AI Tutor Backend

This document gives a short overview on how workflow of the AI Tutor works, when the student/user asks the tutor a question.
The frontend implementation of the AI Tutor can be found [here](../frontend/ai_tutor.md).

---

## Workflow
In the following the "workflow" the AI Tutor goes through to answer the question will be described. 
At corresponding places the used prompt will be linked, but all used prompts can be found [here](#prompts)
> ℹ️ **Info:** Currently we use the ollama Model `llama3:8b-instruct-q4_0` for all LLM requests in the tutor.

### Diagram
![](/images/ai-tutor-diagram.svg)
[Edit Diagram (Image and this link have to be updated manually after editing)](https://mermaid.live/edit#pako:eNp1Vf1v2joU_VeuLE3qNFoehaTFeupEaeirVNaOtJM2IVV-yQWsl9jMdtqxqv_7u84HUGD8gpOce3x87odfWaJTZJxZ_FmgSvBKirkR-VQB_ZbCOJnIpVAOHi0aELb8b8euSFG5fdQo8piR0cqhSve_D-5vPOChcNrEaJ5lgvuguywTufC4ejUmjdk-bhh7zLDcy_2R7Uong3JTWtwb__BH6LVw-CJWHlwv22NMpVgHVCEfPkDscAkdDjGdUqo5uAVCjtaKeU3rbTq-uBhFHCLSZ4DstU5qBUefQC_9SmSQ6MJYvEk_VkGjiELIIg6WeMcV31ET2dqGv1dyymFIgufayN_CY6vPREWElYlbCNwSE3-PH6JxC26j4cPjJGrB3cM_0aQFj18m0fDu-svNj8HlbVTrq5iOG5ETdIVRkFS8q11RXW9PTr7KhBbCJAs40ipbgZw129W8InPwtZHU0IG0DaxCbU40jDl8QyNnq7UlIFQKRVmjSULGbUKG8VrwoPwEzz5UYnqIdoSOdFas9FeW1kGu5vAV5Ca1u2xl3XGQPvuU6yeln0ThFk-29oQW3pOjDUNrnZePG7KSZndXq43DFDJpHegZVcs8JwoLL9ItwCba4J6cMvybyGRK_oLS6hjzpVuVHAexl4XMUlBF_i-arb0MZvjsm8Xhr73AptSuUaHx-whlXygnhfVNUp8UPq3PuYk_XFpV-EF5A-dEmanGPd-1lvKWNGcvB9D7kuzxprG3G6R-xWEmfddRw0-QPEzt5Yp4txJUp6XG74rNfST540Mp7TO9vYcfBO8ORS6gMjJZkLdr2e_VBnxnkpbzwTPRzLPOFAmN5cmthXZVr7b9d9MOF-1SjX9Ri7_4vKRhAtrA52eZor7XVm5SUDL7mcXhStpl5odgI7NSRwWg_iOJrMXmRqaM0_7YYjmaXPhH9uqZpowmYY5TxmmZ4kwUmZuyqXqjMBqxP7TOm0iji_mC8ZnILD0VS1-X9e3TQKhfdLxSSfNMR6J7Y1zdWOXFVbIy_sp-MR52T4LgrNMNg24Q9npBt8VWjHfC85PzMOz2e3_1Tvvh-Vnw1mK_Sx2dk6Df7YT9XhCedoOzs5AiDJmNZqgL5Rjvv_0PkShMxg)

### 1. Sending the message from the frontend
For the first step the user/student has to send the question to the endpoint `sendMessage`:
- This endpoint needs to have the question as a non-empty string
- Optionally for the endpoint is a `courseId` (For question about a course this required)

### 2. Categorizing 
Before answering the question it is preprocessed. Currently the preprocess step only categorizes the question to determine the following workflow for this type of question.
This is done by inserting the question into [this prompt](#categorization) and sending this prompt to Ollama.
The question is then fitted into one of the following categories:
1. `SYSTEM`: A question in this category is about anything in the MEITREX System, e.g. where to find the settings
2. `LECTURE`: This is a question about a specific lecture to further understand a topic
3. `OTHER`: A question of this category does not fit in either system or lecture questions, e.g. 'Give me a recipe for a chocolate cake'
4. `UNRECOGNIZABLE`: A question is only fitted into this category if there are so many typos, that the LLM cannot understand the intent behind the question

> ℹ️ **Info:** Currently only questions of the category `LECTURE` are considered and further processed


### 3. Semantic Search (ONLY LECTURE QUESTIONS)
If the question is categorized as LECTURE, the following steps are performed:
1. Course ID Check
Verify that a courseId was provided.
Ensure the user has access to the course.

2. Fetch Course Content
Query all available content of the course via the ContentService.

3. Semantic Search Execution
To perform the semantic search the contentIds and the question of th euser are send to the `internal_no_auth_semantic_search` endpoint of the DocProcAIService.
This endpoint returns `Document`, `Video` and `Assessment` segments that fit the question.
The returned list is sorted with a similarity score between 0 and 1, where a score closer to 0 means that this segment has content similiar to the question.

4. Validation
Ensures the semantic search does not return an empty list.

5. Relevance List Construction
Concatenate the retrieved content snippets into a numbered list (string).
For Documents the `text` field of the segment is used, which is the text displayed on this page of the document.
**Currently only the text of documents is used** 

6. Answer Generation
The numbered list of contents, together with the question of the user, is then fed into the [prompt](#answering-the-question)
The Ollama model generates an answerto that question that incorporates the found content.

7. Source Tracking
To allow the message to contain links to the segments as sources, the contentIds of these segmebnts are attached to the response.
**Response JSON:**
```json
{
  "answer": "String of the generated answer",
  "sources": ["3fa85f64-5717-4562-b3fc-2c963f66afa6"] 
  //possibly empty list of contentIds (empty for answers of other question categories)
}
```

### 4. Gateway
To build the links and display them in the message of the AI Tutor more information of the MediaRecord is required, thus an additionalTypeDef is added to the gateway
- Use the contentIds from the sources in the `findMediaRecordsByIds` of the MediaService.
- Attach the corresponding media record information to the response.

### 5. Frontend Integration
- Constructs the ULRs for the tutors response with the strcuture `/courses/<courseId>/media/<contentId>`
=> depending on the content type (Document/Video) either the page (`?page=<pageNumber>`) or the startTime (`?videoPosition=<startTime>`) is appended to the URL
=> the source is then displayed using the name of the content and the corresponding addition of page/startTime, e.g. "Introduction Page 3"
>ℹ️ **Info:** In the code, pages are zero-indexed (the first page is 0), but in URLs and the media display, pages are one-indexed (the first page is 1)

## Possible Improvements
1. Thresholding the score of the semanticSearch results, as currently all segments are returned (highest score in tests was approximately 0.6)
2. When answering let the LLM **always** reference the number of the segment it used for this answer 
3. Only generate the links for the sources that specifically had been used in the answer instead of all that were fed into the LLM
4. Add a preprocessing step that generates better search-keys for the semantic search instead of using the whole user question
5. Extend the mutation to use the Chat History to be able answer follow up questions
6. Add the name of the course the user is inside of the preprocessing prompt to better decide if the question is about this course/topic

## Prompts
The used prompts are situated in the `prompt_templates` folder under resources in the `tutor_service` => `src/main/resources/prompt_templates`

### Categorization

<details>
  <summary>Filename: 'categorize_message_prompt'</summary>

  ```txt
  Your job is to categorize a given question into one of four categories: SYSTEM, LECTURE, OTHER, or UNRECOGNIZABLE.
  To do this you must also correct typos and grammar in the question where possible.
  If the corrected version makes the meaning clear, use it to choose the appropriate category.
  Categories:
  1. SYSTEM: Choose this if the question is about the website or app the user is Categories:
  1. SYSTEM: Choose this if the question is about the website or app the user is currently using. For example: "Where do I change my avatar?" These questions relate specifically to the system or platform itself.
  2. LECTURE: Choose this if the question is about understanding or explaining academic content related to a university course. These questions should NOT relate to the website or app. These question are not limited to only a "question" itself, but can also be message that convey the meaning of trying to understand a topic on a deeper level. For example: "What is unsupervised learning" or "Can you give examples for the Chain rule in differentiation"
  3. OTHER: Choose this if the question fits neither SYSTEM nor LECTURE categories and is a general inquiry.
  4. UNRECOGNIZABLE: Choose this if the question is too garbled, incomplete, or contains too many typos or grammar issues to be understood reliably.

  Only use the UNRECOGNIZABLE category when the meaning is truly unclear even after trying to correct it.

  Your response must be a JSON object ONLY (no extra text) with the exact following structure and keys:

  {
    "question": "the original question here",
    "category": "SYSTEM" | "LECTURE" | "OTHER" | "UNRECOGNIZABLE"
  }

  QUESTION:
  {{question}}
  ```
</details>

### Answering the Question

<details>
  <summary>Filename: 'answer_lecture_question_prompt'</summary>

  ```txt
  You are an AI tutor helping a student understand lecture material.

  Below is a question the student has asked, along with excerpts from the lecture that are relevant to the question.
  Please use only the information in the provided lecture content to answer the question as accurately as possible.

  If the information is insufficient to answer, say so and avoid making assumptions.

  Always answer in the same language as the student's question.
  Do not translate or localize technical terms or vocabulary from the lecture content — keep them exactly as they appear.

  Your entire response must be a valid JSON object of the following form:

  {"answer": "your answer here"}

  If it helps clarity, you may refer to specific parts of the lecture content using their assigned numbers (e.g., "[1]", "[2]", etc.).
  ---

  Question:
  {{question}}

  ---

  Relevant Lecture Content:
  {{content}}

  ```
</details>