# User service
This service is dedicated to handling all use cases related to user data.

It plays two primary roles in the MEITREX system:

### 1. GraphQL interface for user metadata

This service acts internally (in our system) as a graphQL endpoint extension for our [Keycloak application](../authentication/keycloak.md) that stores and manages all registered users in a system. This is necessary as Keycloak only supports RESTful request messages. The [gateway service](./gateway-service.md) queries user data over this endpoint.

System-wide permissions (e.g. ability to create new courses) are not handled by this service. These are instead managed by [Keycloak](../authentication/keycloak.md).

### 2. OAuth2 Access Token Management

In addition to acting as a GraphQL interface, the User Service manages **OAuth2 access tokens** for third-party service integrations (such as GitHub). This includes:
- Exchanging authorization codes for access tokens
- Storing token metadata (expiration time, user ID, provider)
- Automatically refreshing expired tokens when a valid refresh token exists
- Providing valid tokens to other services that require authenticated API access on behalf of users

This functionality is crucial for enabling external integrations (e.g., GitHub Classroom) without exposing sensitive authentication logic across the system. Other services can securely retrieve tokens through GraphQL and use them to interact with APIs like GitHub.

A more technical description of the user service and its GraphQL endpoints can be found in our [Github Repository README](https://github.com/MEITREX/user_service#readme).

