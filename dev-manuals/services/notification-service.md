# Notification service

## Data structures
The core responsibility of the notification service is handling all operations related to in-app notifications.  
Next to creating, listing, marking as read, and deleting notifications, this service persists a normalized notification entity and separate recipient rows per user to track delivery status. Notifications are **lightweight abstractions**: they do not contain heavy payloads or media. Instead, they store metadata (title, message, link, source) and reference other services via navigational links.

Notifications can be targeted directly to users or broadcasted to all members of a course. User preferences (per-category toggles) are respected during delivery. If the user settings cannot be retrieved, the service defaults to delivering the notification to avoid missed critical information.

A more technical description of the notification service and its GraphQL endpoints can be found in our [Github Repository README](https://github.com/MEITREX/notification_service#readme).

## Notification types
Currently, our system supports notifications originating from multiple domains (via the shared event model):
1. **Media notifications** (e.g., new material published) from the [media service](./media-service.md).
2. **Content notifications** (e.g., chapter updates) from the [content service](./content-service.md).
3. **Course-related notifications** (e.g., course material) from the [course service](./course-service.md).
4. **Other platform notifications** (e.g., assignments, gamification) as needed.

Beyond the source category and optional course context, the notification service does not store the full domain object. Detailed content remains in the corresponding services and is accessed via the `link` stored with each notification.

## Tracking of delivery and read status
For each targeted user, the service stores a **recipient row** with a status: `UNREAD`, `READ`, or `DO_NOT_NOTIFY` (the latter when user preferences mute a category). Clients can query:
- The **list of notifications** for a user, with the mapped `read` flag.
- The **unread count** for a user.
- Mutations to **mark a single notification** as read or **mark all** as read.
- Mutations to **delete a single notification** or **delete all** notifications.

Ingestion of new notifications happens via domain events emitted by underlying services such as the [media service](./media-service.md), [quiz service](./quiz-service.md), [flashcard service](./flashcard-service.md), [content service](./content-service.md), etc. The notification service consumes these events, persists the notification, materializes recipient rows, and (if applicable) pushes live updates to subscribers. When a notification becomes **orphaned** (no remaining recipients), it is removed to keep storage lean.

## Notification subscriptions
In addition to list queries, the notification service provides a **live subscription** for clients to receive newly created notifications in real time. Subscriptions can be filtered by the authenticated user. Optional filters (such as source category or course) can be layered by the API gateway if required.

A typical subscription flow:
- Clients subscribe to the “notification added” stream for a specific user.
- When an event is ingested and the user is an eligible recipient (and not muted), a `NotificationData` payload is emitted.
- The payload contains: `id`, `title`, `description`, `link`, `createdAt`, and a `read` flag (initially false).

## Management of preferences and recipients
Per-user notification **preferences** (category toggles) are managed by the user/settings domain and queried by the notification service during delivery. If settings are not available at delivery time, the service defaults to delivering the notification to prevent lost information.

Recipient materialization and cleanup:
- For **direct** notifications, the event names explicit user IDs.  
- For **course-wide** notifications, the service resolves course membership via the [course service](./course-service.md) and materializes a recipient row for each member.
- When a user **deletes** a notification, only their recipient row is removed; if no recipient rows remain, the notification entity is also deleted.
- If a user or course is removed by upstream services, corresponding recipient rows are cleaned up as part of standard data hygiene (either via events or periodic maintenance), ensuring referential integrity.

For more information on how other services emit domain events and how users can tune preferences, we recommend reading the corresponding service documentation 
