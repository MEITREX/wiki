# Deletion Concept

Whenever data is deleted, all dependent object need to be updated or removed. In a distributed system, data is spread out over different components (in our case different services) making the deletion of data more challenging and expensive. 

## Data Dependencies
![](/images/simplified_service_dependencies.png)
[Edit Diagram (Image and this link have to be updated manually after editing)](https://mermaid.live/edit#pako:eNqVVNuK2zAQ_RWh5yjEjpMYwy4s2S0UutBN6EvJi2JPbLG27Epy9hLy7x1f4lh2A9s3aebozJkjjU40zCOgAT2k-VuYcGXIj81OEqLLfax4kZAwL5UGpkEdRQhVipB1HSN3d5iVhgupcX1P1gkvDKgKAzIasEgD0tg0WwiNyCVh7MrD2D3ZGh6DbjH1egRZN3wXOfUGE4w8QyQ4u5l-0Bq0zrrcLxTEfqo8Vhhmj9xwwqbMpr-eGcn4biDrtdsagBRX2NRWewvSerGTlm1Z3Y1lWh3aQJirCCVcOCoxDfgg0sa7VpNlCJZVUGCvuNbYZ49sUPlPKT7twi8YGRnwUoJudHcF-3bZ1SqGClG5NshdeSwVXGsRy2z0ch66-EjR0zuoUODrHL2qcg9t7itqryVuSO4DLNGHlOsk5Cpqn_C3y34LZnBlXepL_vWJboiyCDtFCt4wZFu4aWN4930_-mHkhiNPS26gKj-elUHj-lWkKUvhCOlg0PuJQcFR7n-rlhXgH98TyyDbg9KJKPrzYP1gvSls5nR4kE5oBirjIsI_8lSfpyaBDHY0wGUq4sTs6E6eEchLk28_ZEgDo0qYUJWXcUKDA0817soiwo4eBUfVWRctuPyd59nlCI6jydVz8yXXP3MNocGJvtOArVaL6WwxW668meeuXNeZ0A8azP3pfLV05ovFbLZ0fN_zzhP6WbM6U8-Z-47ru9Uhd-F6ExqrqplWIpoCCtuWBsFz3z3_BaFZDao)

In our system, data dependencies can be abstracted into three levels. The top level is the course level. A course contains a set of members and an array of chapters, which are divided into sections and contain content. Whenever a course is removed, all memberhsips, chapters, sections,and content need to also be removed.

The second level can be described as the content level. Content is grouped into Stages that make up a section. For all content, a user can track progress. On basis of tracked progress, a reward-score and skill-level-scores are calculated for each individual member of a course. When a user is deleted or leaves a course, all user data located in the reward-, skill-level-,and content-service are to be removed for the course (or all if user is deleted completely).

On the final level we can cover all different kinds of content. Currently we support media content and three assessment types: flashcard sets, quizzes and assignments. Whenever an assessment is deleted, the corresponding flashcard set, quiz or assignment needs to also be removed. When a question, flashcard or (sub)exercise is deleted, the content and skill-level service are notified, so that the corresponding information in these services can be deleted. Meanwhile media can be re-used in different courses, meaning if media content is removed, the link in the mediarecords on the media-serive need only to be updated and not necessarily removed entirely.

## Deletion events

To communicate changes and deletions of data between services, our services initiate events, broadcasting changes done to their data.
For this we make use of the technology [dapr](../backend/dapr/dapr-pubsub.md) with which we can multicast these event messages on the basis of a publish-subscribe model.

If a object with possible dependent objects is deleted, an event is published to a for the object dedicated topic. The message contains the id of the deleted object and specifies a deletion operation. Other services can subscribe to this topic and react to the deletion of a object. When such an event message is received by a service, it can delete or change all linked objects.
Depending on the dependency level we covered above, these deletion events can have a cascading effect.
E.g. if a course is deleted, a course-change-event is published. The content service receives the message and removes all contents that were linked to the course. the content service then publishes these changes to a different topic. 
The services on the bottom level such as the flashcard, quiz and assignment service will receive the message from the content service and delete all their flashcards, quizzes and exercises that were linked to the deleted content.


## Dapr PubSub

All deletion events follow the same kind of message structure:

```json
{
  "id": "<uuid object>",
  "CrudOperation": "DELETE"
}
```
in case of batch operations:
```json
{
  "ids": "<List of uuids for objects>",
  "CrudOperation": "DELETE"
}
```
As these messages are not only used for deletion operations but also for update operations we specify next the ids of the objects the performed operation. 
A more detailed overview of the different topics and their message contents can be found [here](../backend/dapr/dapr-topics.md).
