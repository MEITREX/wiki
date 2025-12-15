# Dapr Service Topic Overview

![topics.drawio.svg](topics.drawio.svg)

Here we will describe all dapr topics currently available in the system.
This includes

1. a **description** of the topic,
2. Its **interface** description,
3. the content of the **messages**,
4. services that **publish** to the topic,
5. and services that **subscribe** to the topic.

## Events by service

* Course Service
    - Subscribes To: [course-changed](#topic-course-changed), [chapter-changed](#topic-chapter-changed)
    - Publishes: nothing
* User Service
    - Subscribes To: nothing
    - Publishes: nothing
* Content Service
    - Subscribes To: [chapter-changed](#topic-chapter-changed), [content-progressed](#topic-content-progressed), [item-changed](#topic-item-changed)
    - Publishes: [content-changed](#topic-content-changed), [user-progress-updated](#topic-user-progress-updated)
* Media Service
    - Subscribes To: [content-changed](#topic-content-changed)
    - Publishes: [content-progressed](#topic-content-progressed), [media-record-file-created](#topic-media-record-file-created), [media-record-deleted](#topic-media-record-deleted), [content-media-record-links-set](#topic-content-media-record-links-set), [forum-activity](#topic-forum-activity)
* Flashcard Service
    - Subscribes To: [content-changed](#topic-content-changed)
    - Publishes: [content-progressed](#topic-content-progressed), [item-changed](#topic-item-changed), [assessment-content-mutated](#topic-assessment-content-mutated)
* Quiz Service
    - Subscribes To: [content-changed](#topic-content-changed)
    - Publishes: [content-progressed](#topic-content-progressed), [item-changed](#topic-item-changed), [assessment-content-mutated](#topic-assessment-content-mutated)
* Assignment Service
  - Subscribes To: [content-changed](#topic-content-changed)
  - Publishes: [content-progressed](#topic-content-progressed), [item-changed](#topic-item-changed), [assessment-content-mutated](#topic-assessment-content-mutated)
* Reward Service
    - Subscribes To: [user-progress-updated](#topic-user-progress-updated), [course-changed](#topic-course-changed)
    - Publishes: nothing
* Skilllevel Service
    - Subscribes To: [user-progress-updated](#topic-user-progress-updated), [course-changed](#topic-course-changed)
    - Publishes: nothing
* DocProcAi Service
    - Subscribes To: [media-record-file-created](#topic-media-record-file-created), [media-record-deleted](#topic-media-record-deleted), [content-media-record-links-set](#topic-content-media-record-links-set), [assessment-content-mutated](#topic-assessment-content-mutated), [content-changed](#topic-content-changed)
    - Publishes: nothing
* Gamification Service
    - Subscribes To: [user-progress-updated](#topic-user-progress-updated), [content-progressed](#topic-content-progressed), [forum-activity](#topic-forum-activity), [request-hexad-player-type](#topic-request-hexad-player-type), [request-user-skill-level](#topic-request-user-skill-level)
    - Publishes: [user-hexad-player-type-set](#topic-user-hexad-player-type-set)
* Tutor Service
    - Subscribes to: nothing
    - Publishes: [asked-tutor-a-question](#topic-asked-tutor-a-question)
* Assignment Service
    - Subscribes To: [content-changed](#topic-content-changed)
    - Publishes: [content-progressed](#topic-content-progressed), [item-changed](#topic-item-changed), [assessment-content-mutated](#topic-assessment-content-mutated), [student-code-submitted](#topic-student-code-submitted)
* Skilllevel Service
    - Subscribes To: [user-progress-updated](#topic-user-progress-updated), [course-changed](#topic-course-changed), [request-user-skill-level](#topic-request-user-skill-level)
    - Publishes: nothing

## Topic: Course Changed

This topic is used by the Course Service to inform Course-dependant Services of changes done to a Course Entity.

### Interface Description

<dl>
<dt>Name</dt>
<dd>course-changed</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/CourseChangeEvent.java">CourseChangeEvent</a></dd>
</dl>

### Message Content

| Field     | Type | Description                                                                                               |
|-----------|------|-----------------------------------------------------------------------------------------------------------|
| courseId  | UUID | Identifier of a Course Entity                                                                             |
| operation | Enum | Describes which type of CRUD operation was applied to the Course. Available Operations are CREATE, DELETE |

## Topic: Chapter Changed

This topic is used by the Course Service to inform Chapter-dependant Services of changes done to a Chapter Entity.

### Interface Description

<dl>
<dt>Name</dt>
<dd>chapter-changed</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/ChapterChangeEvent.java"> ChapterChangeEvent</a></dd>
</dl>

### Message Content

| Field       | Type        | Description                                                                                        |
|-------------|-------------|----------------------------------------------------------------------------------------------------|
| Chapter IDs | List\<UUID> | A list of Chapter IDs. Each Content is linked to one Chapter ID. Each ID is represented as a UUID. |
| Operation   | Enum        | Describes which type of CRUD operation was applied to the Chapter. Available Operation is DELETE.  |


## Topic: Content Changed

This topic is used by the Content Service to inform Content-dependant Services of changes done to a Content Entity.

### Interface Description

<dl>
<dt>Name</dt>
<dd>content-changed</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/ContentChangeEvent.java"> ContentChangeEvent </a></dd>
</dl>

### Message Content

| Field       | Type        | Description                                                                                                                |
|-------------|-------------|----------------------------------------------------------------------------------------------------------------------------|
| Content IDs | List\<UUID> | A list of Content IDs. A resource can be part of one or multiple Contents. The Content IDs are each represented as a UUID. |
| Operation   | Enum        | Describes which type of CRUD operation was applied to the Content. Available Operations are UPDATE, DELETE                 |

## Topic: Item Changed

This topic is used by the Quiz, Flashcard and Assignment Service to inform Item-dependant Services of changes done to an Item Entity.

### Interface Description

<dl>
<dt>Name</dt>
<dd>item-changed</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/ItemChangeEvent.java"> ItemChangeEvent </a></dd>
</dl>

### Message Content

| Field       | Type        | Description                                                                                                                |
|-------------|-------------|----------------------------------------------------------------------------------------------------------------------------|
| Item ID     | UUID        | Identifier of the Item                                                                                                     |
| Operation   | Enum        | Describes which type of CRUD operation was applied to the Content. Available Operations are DELETE                         |


## Topic: Content Progressed

This topic is used to communicate that a certain content has been completed by a user.

### Interface Description

<dl>
<dt>Name</dt>
<dd>content-progressed</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/ContentProgressedEvent.java">ContentProgressedEvent</a> </dd>
</dl>

### Message Content

| Field          | Type            | Description                                                     |
|----------------|-----------------|-----------------------------------------------------------------|
| userId         | UUID            | The ID of the user associated with the progress log event.      |
| contentId      | UUID            | The ID of the content associated with the progress log event.   |
| success        | boolean         | Indicates whether the user's progress was successful or not.    |
| correctness    | double          | The level of correctness achieved by the user.                  |
| hintsUsed      | int             | The number of hints used by the user.                           |
| timeToComplete | Integer         | The time taken by the user to complete the progress (optional). |
| responses      | List\<Response> | The responses of the user                                       |

## Topic: User Progress Updated

This topic is used to communicate that the content service has processed the update of the user progress.

### Interface Description

<dl>
<dt>Name</dt>
<dd>user-progress-updated</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/UserProgressUpdatedEvent.java">UserProgressUpdated</a> </dd>
</dl>

### Message Content

| Field          | Type                | Description                                                                             |
|----------------|---------------------|-----------------------------------------------------------------------------------------|
| userId         | UUID                | The ID of the user associated with the progress update event.                           |
| contentId      | UUID                | The ID of the content associated with the progress update event.                        |
| chapterId      | UUID                | The ID of the chapter associated with the progress update event.                        |
| courseId       | UUID                | The ID of the course associated with the progress update event                          |
| success        | boolean             | Indicates whether the user's progress was successful or not.                            |
| correctness    | double              | The level of correctness achieved by the user.                                          |
| hintsUsed      | int                 | The number of hints used by the user.                                                   |
| timeToComplete | Integer             | The time taken by the user to complete the progress (optional).                         |
| responses      | List\<itemResponse> | A list with the responses of the user and additional information to the answered items  |

## Topic: Assessment Content Mutated

Raised when the contents of an assessment were changed (e.g. a question of a quiz was added/removed/edited).

### Message Content
| Field                 | Type                 | Description                                                                                            |
|-----------------------|----------------------|--------------------------------------------------------------------------------------------------------|
| courseId              | UUID                 | The ID of the course the assessment which was mutated belongs to.                                      |
| assessmentId          | UUID                 | The ID of the assessment which was mutated.                                                            |
| assessmentType        | Enum                 | One of either [QUIZ, FLASHCARDS, ASSIGNMENT] depending on the type of the assessment.                  |
| textualRepresentation | List<String>         | Textual representations of the tasks of this assessment (and their solutions). For more info see below |

### Textual representation

The purpose of this field is to provide the contents of this assessment in a simple textual format for other services to use
without having to have any assessment-specific code.

It is a list of Strings, so if an assessment consists of multiple tasks/parts, each part can have its own textual representation.
If the assessment only consists of one task or if the nature of the assessment makes it hard to split it up into concrete parts, it 
is acceptable to only add a single textual representation String to the list, which covers the whole assessment.

If a service can not/does not want to provide a textual representation of the assessment, the list can be left empty.

There is no exact formatting guideline for the textual representation. Its purpose is to just provide a very basic means to
fetch assessments' contents in textual form, e.g. for performing a search or similar purposes.

The textual representation can contain spoilers, so it should never be shown to students.

A good textual representation may look like the following for example, for a quiz:

String 1:
```
Question: How tall is the Eiffel Tower?
Correct Answer: 330m
```

String 2:
```
Question: What colors make up the French flag?
Correct Answer: blue, white, and red
```

String 3:
```
Task: Fill in the gaps in the text:
King Luis XIV famously said: "L'[etat], c'est [moi]!"
```

## Topic: Media Record File Created

Raised when a new file is uploaded for a media record.

### Message Content
| Field          | Type                 | Description                                                                   |
|----------------|----------------------|-------------------------------------------------------------------------------|
| mediaRecordId  | UUID                 | The ID of the media record for which a file was uploaded.                     |

## Topic: Media Record Deleted

Raised when a media record is deleted

### Message Content
| Field          | Type                 | Description                                                                   |
|----------------|----------------------|-------------------------------------------------------------------------------|
| mediaRecordId  | UUID                 | The ID of the media record which was deleted.                                 |

## Topic: Content Media Record Links Set

Raised when a `MediaContent`'s media record links are set/changed.

### Message Content
| Field          | Type                 | Description                                                                   |
|----------------|----------------------|-------------------------------------------------------------------------------|
| contentId      | UUID                 | The ID of the content whose media record links were set.                      |
| mediaRecordId  | List<UUID>           | List of the IDs of the media records which were linked to the content.        |


## Topic: Forum Activity

| Field    | Type          | Description                                       |
|----------|---------------|---------------------------------------------------|
| forumId  | UUID          | The ID of the forum where the activity happened.  |
| courseId | UUID          | The ID of the course where the activity happened. |
| userId   | UUID          | The ID of the user that did the activity.         |
| activity | ForumActivity | The kind of activity that happened.               |

## Topic: Asked Tutor a Question

Raised when the user asks the AI Tutor a question, BUT only after the question has been categorised

| Field    | Type          | Description                                                                                      |
|----------|---------------|--------------------------------------------------------------------------------------------------|
| courseId | UUID          | The ID of the course the question is about happened.                                             |
| userId   | UUID          | The ID of the user that asked the question.                                                      |
| category | TutorCategory | The Category the question fits into (SYSTEM, LECTURE, OTHER, UNRECOGNIZABLE, ERROR, CODE_FEEDBACK)|

## Topic: User Hexad Player Type Set

Raised when the player type is saved inside of gamification_service. This event is sent when a user's Hexad player type is set, updated, or requested for the first time.

### Interface Description

<dl>
<dt>Name</dt>
<dd>user-hexad-player-type-set</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/meitrex/common/event/UserHexadPlayerTypeSetEvent.java">UserHexadPlayerTypeSetEvent</a></dd>
</dl>

### Message Content

| Field                 | Type                          | Description                                                    |
|-----------------------|-------------------------------|----------------------------------------------------------------|
| userId                | UUID                          | The ID of the user whose Hexad player type was set.            |
| playerType            | HexadPlayerType               | The Hexad player type of the user (ACHIEVER, PLAYER, SOCIALISER, FREE_SPIRIT, PHILANTHROPIST, DISRUPTOR). |
| playerTypeScores      | Map<HexadPlayerType, Integer> | Map of all player types to their respective scores.            |

## Topic: Request Hexad Player Type

Used to request a user's Hexad player type from the gamification service. The gamification service will respond with a [user-hexad-player-type-set](#topic-user-hexad-player-type-set) event.

### Interface Description

<dl>
<dt>Name</dt>
<dd>request-hexad-player-type</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/meitrex/common/event/RequestHexadPlayerTypeEvent.java">RequestHexadPlayerTypeEvent</a></dd>
</dl>

### Message Content

| Field  | Type | Description                                                |
|--------|------|------------------------------------------------------------||
| userId | UUID | The ID of the user for which the player type is requested. |

## Topic: Request User Skill Level

Used to request a user's skill levels from the skilllevel service. The skilllevel service will respond with a user-skill-level-changed event.

### Interface Description

<dl>
<dt>Name</dt>
<dd>request-user-skill-level</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/meitrex/common/event/RequestUserSkillLevelEvent.java">RequestUserSkillLevelEvent</a></dd>
</dl>

### Message Content

| Field  | Type | Description                                              |
|--------|------|----------------------------------------------------------|
| userId | UUID | The ID of the user for which skill levels are requested. |

## Topic: Student Code Submitted

Raised when a student submits code for an assignment. This event is triggered by the assignment service when it loads and processes a student's code submission from their repository.

### Interface Description

<dl>
<dt>Name</dt>
<dd>student-code-submitted</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/meitrex/common/event/StudentCodeSubmittedEvent.java">StudentCodeSubmittedEvent</a></dd>
</dl>

### Message Content

| Field              | Type             | Description                                                                 |
|--------------------|------------------|-----------------------------------------------------------------------------|
| studentId          | UUID             | ID of the student who submitted the code.                                   |
| assignmentId       | UUID             | ID of the assignment for which the code was submitted.                      |
| courseId           | UUID             | ID of the course the assignment belongs to.                                 |
| repositoryUrl      | String           | URL of the Git repository containing the submitted code.                    |
| commitHash         | String           | Git commit hash identifying the exact submission.                           |
| submittedAt        | OffsetDateTime   | Timestamp when the code was submitted.                                      |
| submittedFiles     | Map<String, String> | Map of file paths to their contents. Keys are relative file paths within the repository, values are the file contents as strings. |


