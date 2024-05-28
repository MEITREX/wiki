# Skill level service
In our application, we can find two different scoring systems: The [reward system](./reward-service.md) and the skill level system.
This service handles the skill level system of MEITREX.
The general idea of skill levels is to motivate students to learn repeatedly. Six different skill types are implemented according to Bloom's Taxonomy:
- Remember
- Understand
- Apply
- Analyze
- Evaluate
- Create
For more information on how all these skill level scores are calculated and the description of the different skill types we recommend reading our [documentation on the scoring system](../gamification/Scoring%20System.md).
This service offers GraphQL endpoints to query the current skill level scores of a user.
We also use skill types in the [content service](./content-service.md) to make content suggestions to the user based on selected skill types.

## Score calculation and information gathering
Calculations of the skill level of a user are triggered via a user progress event message.
A user progress event is triggered each time a user progresses content. This is forwarded from the [content service](./content-service.md). The event contains all informations necessary to calculate the new skill Levels. 


A more technical description of the skill-level service and its GraphQL endpoints can be found in our [Github Repository README](https://github.com/MEITREX/skilllevel_service#readme).