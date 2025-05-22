# Notification Service
2025-05-14
## Context
The user should get informed about different events happening in the system. The events can get caused by different Services.
The Notification Service creates a notification for every event, displays it and saves the notifications information until the user deletes it.
## Options Considered
- A: Different Services send information about events to the Notification Service
- B: The Notification Service asks the other services repeatedly if an event has happened
## Decision - Option A
- Work is only done when needed, Option B causes a lot of redundant communication
- Which Option B the event needs to be stored until the notification service asks for it. Option A surpasses this problem
- It is to note that when implementing a functionality that creates a notification, it is important to add the event message to the Notification Service (and not forget it)