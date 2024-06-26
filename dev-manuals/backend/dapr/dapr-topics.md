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

| Service            | Subscribes to                                                                                      | Publishes                                                                                          |
|--------------------|----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Course service     |                                                                                                    | [course-changed](#topic-course-changed) <br>[chapter-changed](#topic-chapter-changed)              |
| User Service       |                                                                                                           |                                                                                                    |
| Content Service    | [chapter-changed](#topic-chapter-changed)<br>[content-progressed](#topic-content-progressed)<br>(#topic-course-changed) [item-changed](#topic-item-changed)      | [content-changed](#topic-content-changed)<br>[user-progress-updated](#topic-user-progress-updated) |
| Media Service      | [content-changed](#topic-content-changed)                                                          | [content-progressed](#topic-content-progressed)                                                    |
| Flashcard Service  | [content-changed](#topic-content-changed)                                                          | [content-progressed](#topic-content-progressed)<br>[item-changed](#topic-item-changed)                                            |
| Quiz Service       | [content-changed](#topic-content-changed)                                                          | [content-progressed](#topic-content-progressed)<br>[item-changed](#topic-item-changed)                                            |
| Reward Service     | [user-progress-updated](#topic-user-progress-updated)<br>[course-changed](#topic-course-changed)   |                                                                                               |
| Skilllevel Service | [user-progress-updated](#topic-user-progress-updated)<br>[course-changed](#topic-course-changed) [item-changed](#topic-item-changed)|                                                         |

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

### Involved Services

<dl>
<dt>Publishers</dt>
<dd><ul>
<li>Course service</li>
</ul></dd>
<dt>Subscribers</dt>
<dd><ul>
<li>Reward Service</li>
<li>SkillLevel Service</li>
</ul></dd>
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

### Involved Services

<dl>
<dt>Publishers</dt>
<dd><ul>
<li>Course service</li>
</ul></dd>
<dt>Subscribers</dt>
<dd><ul>
<li>Content Service</li>
</ul></dd>
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

### Involved Services

<dl>
<dt>Publishers</dt>
<dd><ul>
<li>Content service</li>
</ul></dd>
<dt>Subscribers</dt>
<dd><ul>
<li>Media Service</li>
<li>Flashcard Service</li>
<li>Quiz service</li>
</ul></dd>
</dl>

### Message Content

| Field       | Type        | Description                                                                                                                |
|-------------|-------------|----------------------------------------------------------------------------------------------------------------------------|
| Content IDs | List\<UUID> | A list of Content IDs. A resource can be part of one or multiple Contents. The Content IDs are each represented as a UUID. |
| Operation   | Enum        | Describes which type of CRUD operation was applied to the Content. Available Operations are UPDATE, DELETE                 |

## Topic: Item Changed

This topic is used by the Quiz and Flashcard Service to inform Item-dependant Services of changes done to an Item Entity.

### Interface Description

<dl>
<dt>Name</dt>
<dd>item-changed</dd>
<dt>PubSub-Name</dt>
<dd>gits</dd>
<dt>Java class</dt>
<dd><a href="https://github.com/MEITREX/common/blob/main/src/main/java/de/unistuttgart/iste/gits/common/event/ItemChangeEvent.java"> ItemChangeEvent </a></dd>
</dl>

### Involved Services

<dl>
<dt>Publishers</dt>
<dd><ul>
<li>Flashcard service</li>
<li>Quiz service</li>
</ul></dd>
<dt>Subscribers</dt>
<dd><ul>
<li>SkillLevel Service</li>
<li>Content Service</li>
</ul></dd>
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

### Involved Services

<dl>
<dt>Publishers</dt>
<dd><ul>
<li>Media service</li>
<li>Flashcard service</li>
<li>Quiz service</li>
</ul></dd>
<dt>Subscribers</dt>
<dd><ul>
<li>Content service</li>
</ul></dd>
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

### Involved Services

<dl>
<dt>Publishers</dt>
<dd><ul>
<li>Content service</li>

</ul></dd>
<dt>Subscribers</dt>
<dd><ul>
<li>Reward service</li>
<li>Skilllevel service</li>
</ul></dd>
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