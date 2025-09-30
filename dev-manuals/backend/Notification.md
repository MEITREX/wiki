# Send Notifications from your service

Send user-facing notifications by publishing a **NotificationEvent** to the platform pub/sub.  
No direct call to the Notification Service is needed.

---

## 1) Dependencies (common + optional SDK)

**Common library (events & enums)**
- Package imports you’ll use:
  - `de.unistuttgart.iste.meitrex.common.event.NotificationEvent`
  - `de.unistuttgart.iste.meitrex.common.event.ServerSource`
  - `de.unistuttgart.iste.meitrex.common.dapr.TopicPublisher`
- Dependency (example coordinates; use your internal artifact):
  
    Gradle:
        implementation("de.unistuttgart.iste.meitrex:meitrex-common:1.4.9")

**(Optional) Dapr Java SDK** (publish via SDK instead of raw HTTP)
- `io.dapr:dapr-sdk:1.9.0`

---

## 2) Payload shape: `NotificationEvent`

Either provide **userIds** (direct recipients) **or** a **courseId** (broadcast).
When courseId is included, course title will be added automatically.
If the settings service is down, delivery **fails open** (users still get the notification but won't be queried by frontend).

    {
      "userIds": ["<uuid>", "..."],   // XOR courseId
      "courseId": "<uuid>",           // XOR userIds
      "serverSource": "MEDIA",        // use ServerSource enum
      "title": "New material uploaded",
      "message": "Lecture 02 slides are available",
      "link": "/courses/<courseId>/media/<contentId>?selectedDocument=<mediaId>"
    }

Notes:
- Use your real front-end path in `link` (document/video/quiz/flashcards, etc.).
- If both `userIds` and `courseId` are empty → the event is ignored.

---

## 3) Publish

**Java + Dapr SDK**

    import de.unistuttgart.iste.meitrex.common.dapr.TopicPublisher;
    import de.unistuttgart.iste.meitrex.common.event.ServerSource;
    import lombok.RequiredArgsConstructor;
    import org.springframework.stereotype.Service;
    
    import java.util.UUID;
    
    @Service
    @RequiredArgsConstructor
    public class MyDomainService {
    
      private final TopicPublisher topicPublisher;
    
      public void sendNotification(UUID courseId) {
        String link = "/courses/" + courseId;
        topicPublisher.notificationEvent(
            courseId,                 // course broadcast
            null,                     // OR: a list of userIds
            ServerSource.MEDIA,       // choose the enum matching your service
            link,
            "New material uploaded",
            "Lecture 02 slides are available"
        );
      }
    }

> Replace `messagebus` and `notifications` with your actual **pubsub component** and **topic**.

---

## 4) Quick verification

GraphQL (via gateway):

    query {
      currentUserInfo {
        id
        notificationUnreadCount
        notifications { id title read }
      }
    }

If everything is wired, unread count increases and the new item appears.

---

## Cheatsheet

- Choose **one**: `userIds` **or** `courseId`.
- Use `ServerSource` enum (e.g., `MEDIA`, `CONTENT`, `ASSIGNMENT`, …).
- `link` must point to a valid front-end route.
- Pub/sub names and ports come from your Dapr setup (`DAPR_HTTP_PORT`, component name, topic name).
