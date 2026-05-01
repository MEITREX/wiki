# Introduction
This is a research project that aims to integrate automated UML assessment into Meitrex. The system will incorporate Hylimo to enable the creation and submission of UML assignments. Student solutions will then be automatically evaluated based on reference solutions provided by tutors.

**What is HyLiMo?** Hylimo is a hybrid diagramming tool that enables users to create and edit diagrams, such as UML diagrams, by using a combination of textual code (a domain-specific language) and a graphical editor. Changes made to either the code or the visual representation are automatically synchronised, ensuring both views always remain consistent.
This paper consists of:
- *Requirements*
- Architecture and Decisions
- HyLiMo Integration
- Automated UML Assessment
- Evaluation
- Functionalities (Developres + Lecturer Guide)
- Future Work


- prompt wie bei ai tutor orientieren

# Requirements
Based on the objectives of this research project, the following section defines the high-level requirements for integrating automated UML assessment into Meitrex using HyLiMo.


- Creating a UML assessment (Task and Solution)
- Working on a UML assessment
- Saving a UML assessment solution
- Submitting a UML assessment
- Redoing a UML assessment (if possible)
- Evaluate a UML diagram
- View Submitted UML Assessment
- Lecturer Views and Manages Student Submissions

For Use Cases checkout: https://docs.google.com/document/d/1zKl2Vpk7k4WUOQumee5xeIibMjcerCa-Pp7XphvoagM/edit?tab=t.0


# Architecture and Decisions

## Systemarchitecture
Show MEITREX + HyLiMo System Context --> Component Diagrams



## Domain Model
The diagram shows the domain model for the UML assignment. An UmlExercise is the task created by the tutor. For each exercise, a student has a UmlStudentSubmission, which contains all their attempts. Each submission can include multiple solutions (UmlStudentSolution). Every solution contains a UML diagram and can receive feedback with points and comments.

![Domain Model](img/class_diagram.png)


## HyLiMo Integration
One of the main tasks was the integration of HyLiMo into MEITREX. First, all required components had to be identified. The needed HyLiMo code was originally built using Vite. This code served as the basis for the migration into our Next.js (React) project. As part of this process, the application had to be adapted and converted from the Vite setup to the Next.js architecture.
- TBD: HyLiMo Architektur Diagram


## Automated UML Assessment
The second major task was the automatic evaluation of UML diagrams based on the tutor’s solution.

The sequence diagram illustrates, in a simplified manner, how a student’s UML solution is processed and evaluated across the system. First, the student creates and submits a UML diagram in the frontend, which then transforms the diagram code into a structured semantic model. This model is sent to the backend, where the UmlEvaluationService delegates the evaluation. The backend interacts with a large language model (LLM) in two steps: it first performs an analysis by comparing the student’s semantic model with the tutor’s reference model, and then conducts a grading step that generates points and textual feedback based on the analysis results. Finally, the computed feedback is returned through the backend to the frontend and presented to the student


![Sequence Diagram of the Evaluation](img/sequence_eval.png)


### Prompt Engineering

<details>
  <summary>Filename: 'uml_analysis'</summary>

```txt
### ROLE
You are an Experienced Software Engineering Professor and UML Evaluator.
Your task is to compare the **Reference Solution** against the **Student Submission**, but you must prioritize *theoretical correctness* and *semantic meaning* over strict syntactic matching.

### EQUIVALENCE RULES (CRITICAL - DO NOT PENALIZE FOR THESE)
1. **Data Types:** Treat semantically similar types as identical (e.g., `int` == `Integer`, `String` == `string`, `Long` == `long`, `boolean` == `Boolean`).
2. **Association Labels:** Focus on the *meaning* of the relationship, not the exact string. (e.g., "owns", "has", "contains", and "is part of" are effectively equivalent if the multiplicity and direction are correct).
3. **Architectural Variations:** Students may solve domain problems slightly differently.
   - Example 1: Using an `Enum` for a property vs. a dedicated Class with constraints.
   - Example 2: Using an `abstract class` instead of an `interface`.
   - If the student's alternative logically fulfills the same domain requirement as the reference, accept it as CORRECT.

### DATA TO ANALYZE
- **Reference Solution:**
---
{{tutorModel}}
---
- **Student Submission:**
---
{{studentModel}}
---

### CATEGORIZATION STRICTNESS
- **correctElements:** List all correctly implemented details. If a student used an acceptable equivalent (e.g., `int` instead of `Integer`), list it here as correct.
- **missingElements:** List items that are COMPLETELY absent and have no logical equivalent in the student's code.
- **semanticErrors:** List items that are actively WRONG (e.g., a composition used where an inheritance was clearly required, or fundamentally backward multiplicities).

### CRITICAL OUTPUT RULES
- Do NOT deduct points or list errors for layout attributes (e.g., `pos`, `vdist`, `layout`).
- Output ONLY valid JSON.
- Every element analyzed MUST be present in exactly one of the three arrays.

**Output Format (JSON):**
{
"correctElements": ["List specific correct elements and accepted equivalents."],
"semanticErrors": ["List actively INCORRECT structural logic."],
"missingElements": ["List completely ABSENT elements."],
"isSemanticallyValid": <boolean>,
"analysisSummary": "A concise summary derived strictly from the lists above."
}
```

</details>

The 'uml_analysis' evaluates student UML diagrams against a reference solution by focusing on semantic correctness. It identifies correct elements, missing parts, and real structural errors, and outputs a structured JSON analysis.


<details>
  <summary>Filename: 'uml_grading'</summary>

```txt
### ROLE
You are a supportive Software Architecture Tutor. Your goal is to transform technical analysis into constructive HTML feedback and calculate a final grade based strictly on the provided rubric.

### GRADING LOGIC (FOLLOW STRICTLY)
- **Total Possible:** {{maxPoints}} points.
- **Passing Threshold:** {{passingThreshold}} points.
- **Reference Rubric:** {{gradingRules}}

**Calculation Hierarchy:**
1. Start at {{maxPoints}} points.
2. Look at the items in `semanticErrors` and `missingElements`.
3. For EACH error, find the corresponding penalty in the **Reference Rubric**.
4. Subtract the penalty from the current score. (e.g., If the rubric says "-0.25P per missing class" and there are 2 missing classes, subtract 0.5P).
5. DO NOT deduct points for anything not explicitly listed in the errors arrays.
6. If the final score drops below 0, set it to 0.

### PEDAGOGICAL GUIDELINES
1. **The "Sandwich" Method:** Start with specific praise (correctElements), address gaps (semanticErrors/missingElements), and end with motivation.
2. **Spoiler Policy:** The 'Show Solution' flag is: {{showSolution}}
   - **If FALSE:** Provide Socratic hints (e.g., "Check the relationship multiplicity."). DO NOT give the exact answer.
   - **If TRUE:** You may explicitly state the correct answer.
3. **Mandatory Sign-off:** You MUST conclude the HTML string exactly with: `<br><br>Best Regards,<br>Your AI Tutor`

### DATA FOR REPORT
- **Valid Graph:** {{isValid}}
- **Things Done Well:** {{correctElements}}
- **Logic Errors:** {{semanticErrors}}
- **Missing Items:** {{missingElements}}

### TECHNICAL CONSTRAINTS
- **Output Format:** STRICT JSON containing an HTML string and an integer/float.
- **Allowed HTML:** `<b>`, `<i>`, `<p>`, `<br>`, `<ul>`, `<li>`. No Markdown, no CSS.
- **No Score in Text:** DO NOT state the points inside the HTML feedback string.

### OUTPUT SCHEMA
{
"feedback": "<p>Your HTML feedback here...</p><br><br>Best Regards,<br>Your AI Tutor",
"points": <calculated_number>
}
```

</details>

The 'uml_grading' uses the analysis output to generate constructive HTML feedback and calculate a final score. It applies strict grading rules, provides pedagogical feedback, and ensures a structured JSON result with points and feedback.


# Evaluation
## Goals
The goal of this evaluation is to assess how accurately the automated UML assessment system can replicate human grading. In particular, we investigate whether the system can produce scores comparable to lecturer evaluations and how different large language models influence the assessment quality.
## Setup
For the evaluation, we used the UML diagrams that students had submitted in previous lecture assignments. These were originally graded by by human lecturers.


To enable automated processing, the student models were converted into Hylimo code. We used tutor-provided submissions that achieved full marks as reference solutions, representing correct and complete UML models.


## Methodology
An evaluation script was developed to automatically assess the converted UML models against the reference tutor solutions. In addition, multiple large language models were tested to compare their performance in grading UML diagrams. Each submission was evaluated five times to account for variance in the grading results.


## Metrics
The evaluation is based on the following metrics:
Agreement between automated scores and lecturer grades
Performance differences between the tested LLMs
...
## Result
TBD


# Functionalities
## User Guide for Lecturer
### Introduction
Tutors can create and provide model solutions for UML assignments within Hylimo. Student submissions are then automatically evaluated against the tutor solution.
For more information about HyLiMo check out: https://hylimo.github.io/docs/docs.html


#### Functionalities
#### Create UML assignment
UML assignments can be created in the same way as other assignments

![Create Uml Assignment](img/create_uml_assignment.png)

When selecting 'UML Assignment', a pop-up window opens where the assignment can be created.

![Pop-Up Create Uml Assignment Metadata](img/create_uml_assignment_metadata.png)

In the pop-up window, similar to other assignments, the metadata can be filled in. Below that, the UML-specific fields are available.

![Pop-Up Create Uml Assignment Uml Data](img/create_uml_assignment_uml_data.png)

The UML Assignment creator includes a rich text editor where the task description can be written. In addition, grading criteria can be defined to support the LLM-based evaluation of the diagram.
In the settings, you can set the total number of points and the passing threshold, as well as choose whether the model solution should be displayed. The tutor can also provide a HyLiMo reference solution, which is then used as the basis for the LLM’s evaluation.


#### UML Assignment Overview
When a lecturer selects the assignment, they will see an overview of the entire assignment with all information related to the created task. They can then edit the assignment, which opens a pop-up window where changes can be made.

![UML Assignment Overview](img/lecturer_exercise_overview.png)

They can also navigate to the submissions tab to get an overview of the solutions submitted by students.
- TBD Bild

## User Guide for Student
### Introduction
For UML assignments, students complete tasks within Hylimo by creating their own UML models. Their submissions are automatically evaluated, allowing them to receive feedback.
For more information about HyLiMo check out: https://hylimo.github.io/docs/docs.html

#### Functionalities
#### Work on UML Assessment
Students can open the UML assignment just like any other assignment. Once opened, they can work on it using the built-in editor.


![Work on Uml Assignment Uml Data](img/students_uml_assignment.png)


At the top, you can see the task description. Below that, there are options to move between attempts, save the work, or submit the solution. After submitting, a new attempt can be started, either empty or based on a previous one. The main part of the screen is the Hylimo editor, where students work on their solution. The editor can also be opened in full-screen mode for easier editing.

![Work on Uml Assignment Uml Data](img/hylimo_editor_fullscreen.png)


#### UML Evaluation
