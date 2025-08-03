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

## GitHub OAuth Configuration

To enable GitHub access token handling, the User Service must be configured with valid OAuth2 credentials for GitHub.

These credentials are used to:
- Exchange authorization codes for access tokens
- Refresh tokens when needed
- Authenticate GitHub API requests on behalf of users

| Environment | github.clientId      | github.clientSecret | Description                                        |
|-------------|----------------------|---------------------|----------------------------------------------------|
| dev         | Iv23liynxdcJafLw0ptQ | *dev secret*        | Used for local development and Docker environment. |
| prod        | *secret*             | *secret*            | Used for production.                               |

> **Note:** For development don’t forget to initialize `NEXT_PUBLIC_GITHUB_CLIENT_ID` in the frontend with `Iv23liynxdcJafLw0ptQ` as an environment variable.

The MEITREX-TEST GitHub App has already been created for development. Ask a team member or your supervisor for the dev secret.

However, if you want to create your own GitHub App for development, follow the steps below (you’ll need access to your GitHub Organization).

### Setting up GitHub App Credentials 

To set up a Github App (for dev or prod), follow these steps:

1. Go to Organization Settings.
2. Navigate to Developer Settings.
3. Click on "GitHub Apps".
4. Click "New GitHub App".
5. Fill the following fields with the provided values:
   - User authorization callback URL: http://localhost:3005/oauth/callback (for development, replace with production URL in production)
   - Repository permissions: 
     - Actions: Read-only
   - Where can this GitHub App be installed? Any account
6. Click "Create GitHub App".
7. Generate a new client secret and copy it with the client ID to the User Service configuration.
8. Copy the Github App name and set the Organization name in the Assignment Service configuration.
9. Set the client ID in the frontend environment variable `NEXT_PUBLIC_GITHUB_CLIENT_ID`.