# Assignment Service

Assignments are graded assessments that students work on outside of MEITREX. Each assignment consists of exercises (and optionally subexercises) and contributes to evaluating a student's skill level.

Assignments can be imported from external systems or created and managed directly within MEITREX.

**Note:** Currently, only **code assignments** can be partially created (need to be synced from GH Classroom) and managed via the MEITREX frontend.

---

## Assignment Types

| Assignment Type  | Description                                                                        |
|------------------|------------------------------------------------------------------------------------|
| Exercise Sheet   | Done at home or during tutorials at university. Usually graded manually by tutors. |
| Physical Test    | Any form of written or in-person exam. Also manually graded.                       |
| Code Assignment  | GitHub-based programming assignments. Graded via GitHub.                           |

- **Exercise Sheet** and **Physical Test** are imported from an external system (TMS) and cannot be edited or graded within MEITREX yet.
- **Code Assignments** are created in GitHub Classroom, synced with MEITREX and graded via GitHub.

## Tracking a user's progress

The assignment service tracks how much of an assignment a student has completed and how successfully:

- For each assignment, the system calculates a *success* value based on the achieved credits versus required percentage.
- *Correctness* is derived from the ratio of achieved to total credits.
- For code assignments, results are fetched from GitHub Classroom and displayed in MEITREX.
- When grading is complete, the service emits a `ContentProgressedEvent`, which informs the content service and triggers updates across other services (e.g., skill level, rewards).

## GitHub Classroom Integration

To enable integration with GitHub Classroom, the **Assignment Service** requires the specification of a GitHub organization. This organization is where code assignments are created, graded, and synced from.
It is defined using the property: `github.organization_name`

The organization used varies depending on the environment:

| Environment | GitHub Organization | Description                                       |
|-------------|---------------------|---------------------------------------------------|
| `dev`       | `MEITREX-TEST`      | Used for local development and Docker environment.|
| `prod`      | `MEITREX`           | Used for production.                              |
