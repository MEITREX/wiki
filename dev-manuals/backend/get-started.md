# Get Started

This will contain a description on how get started to work as a backend dev.
If you don't want to develop and only run the service, there is docker compose file provided.

## Prerequisites

- Java

  We use Java 21. You can download the JDK [here](https://www.oracle.com/java/technologies/downloads/#java21).

- Git

  We use Git as version control system. You can download it [here](https://git-scm.com/downloads). To be able to push and pull from GitHub, you will need to set up an SSH key. You can find a guide [here]  (https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

- IDE

  We use IntelliJ IDEA as IDE. You can download it [here](https://www.jetbrains.com/idea/download/). As a student you can get a free license for the Ultimate Edition [here](https://www.jetbrains.com/community/education/#students). You can enable auto reload, where when you make changes in the code, the server will automatically reload the code. [Here](https://dev.to/imanuel/auto-reload-springboot-in-intellij-idea-1l65) is a short guide how to enable it.

- Gradle

  We use Gradle as build tool. You can download it [here](https://gradle.org/install/).

- Docker

  We use Docker to run our microservices and the database. You can download it [here](https://www.docker.com/products/docker-desktop).

# Setting Up the Dev Environment
## Cloning

This repository uses git submodules to be able to retrieve the repositories of all services into this repository when cloning. To do this, do the following:

1. As with any repo, clone it using `git clone https://github.com/MEITREX/backend.git`
2. Move into the repository (`cd backend`)
3. Initialize the submodules using `git submodule init`
4. Pull the submodules using `git submodule update`

## Deployment

The local deployment of the backend is done in a few simple steps:
1. Start docker (desktop)
2. Create a network for the dapr sidecars to communicate: `docker network create dapr-network`
3. Execute the .bat or .sh file found under ./backend/compose.bat using `compose.bat up --build` (re-builds the containers) or `compose.bat up` (starts the containers without re-building if they already exist).
  - See [##`docprocai` Service Taking Forever to Build]

## Debugging/ Running Selected Services Locally

To facilitate easy debugging of services, the docker containers are set up to expose all important ports to the host machine. This can easily be seen in Docker Desktop. Port mappings can also be
found [on the wiki](https://meitrex.readthedocs.io/en/latest/dev-manuals/backend/Ports.html).

This makes it possible for you to start a service in your favorite IDE (where you can set breakpoints etc.) and for that service to hook into the rest of the system.

We'd advice you to create a branch in the backend repo to aggregate (api) changes across different repos via git submodules.

* At first, stop the MEITREX docker container and remove the set of services you want to run locally from the `compose.bat`/`compose.sh` list of paths to the submodule's `docker-compose.yml`s

If the service to be run locally is the frontend:

* Start the docker containers again (as described above) and follow the instructions provided in the frontend repos `README.md`

Else:

* Navigate to the `graphql_gateway` submodule, open its `docker-compose.yml` and adjust the URLs of the set of services you want to run locally in the `environment` section as follows: http://~~app-\<service-name\>~~host.docker.internal:\<service-port\>/graphql
* Rebuild & restart the docker services via `compose.bat`/`.sh up --build`. The `gateway` container should now restart frequently because it can't reach the services which are supposed to be run locally. This is desired behavior, since it now tries to fetch the graphql api schemas of the services which URLs have been modified from your machines `localhost` and the respective service-ports.
* Make sure the Spring Boot dev profile's database port is set correctly
* Start the set of service you want to run locally on your machine using the dev profile, which can either be selected in the IDEA Ultimate  "Run Configurations", the `application.properties` file or be passed to gradle as command line argument: `gradle bootRun --args="--spring.profiles.active=dev"`
<!-- TODO python? -->
* The Dapr PubSub Events do not work when debugging in this way

# Some Hints/Common Issues

## `docprocai` Service Taking Forever to Build

Since the `docprocai` service leverages a huge LLM for its functions and this LLM is being pulled on every build of the container, the container might take a while to start. To prevent the pull, the following changes can be made:

1. Create a folder with the path `llm_data/models` in the `docprocai` service repo
2. Navigate into it and clone e.g. `https://huggingface.co/Alibaba-NLP/gte-large-en-v1.5` (should take a while)
  - If the download of the model file (managed by git large file system) fails, you can download it manually from the git web UI and move it into the cloned huggingface.co repo
  - You can go even further and disable the usage of the models which might speed up the build. To do this, set the `enabled: true` values for the AI features to `false`  in the `config.yaml` file
3. Build the container

## "database <service_name> does not exist" FATAL Log Messages in `database` Container

When creating the `database` container, it might happen that the databases for the services aren't created properly. This is due to Windows file-permission overwriting the executable but necessary to run the script in the docker container's file system.
To fix this, use a DB client of your choice (e.g. pgadmin4), establish a connection to the database container's postgres server on port `5432`. Then, unravel the tree entries "Servers" > "\<database-container\>" > "Databases" and right-click on "Create" > "Database...".
Create the `content_service`, `course_service`, `docprocai_service`, `flashcard_service`, `gamification_service`, `media_service`, `quiz_service`, `reward_service`, `skilllevel_service` & `user_service` databases by adding the strings to the "Database" entry in the "General" tab and add the "Privilege" entries ("PUBLIC", "Tc", "root") & ("root", "CTc", "root") in the "Security" tab.
After doing these adjustments, the issue should be resolved.

## Observing/ Testing the GraphQL API

Requires a local `python3`, `pip` & `pyperclip` installation.

1. Make sure the gateway service is up & running, then open `localhost:8080`. This should display a GraphQL Mesh "playground-like" query tool.
2. Open the folder-icon on the left side bar to use assemble your query easily via the schemas API-endpoint-tree.
  - If an ID is a necessity for the query, use a DB query tool of your choice (e.g. pgadmin4) to fetch it.
3. In the `backend` repository, run the `demo_scripts/get_session_token.py` script with the `--local` argument. The script also accepts the `--username` and `--password` argument to speed up recalls. The generated token should be valid for a view minutes.
4. Back in the GraphQL Mesh playground UI, switch to the "Header" tab at the bottom of the screen. Enter the following JSON:

```json
{
  "Authorization": "Bearer <generated-keycloak-token>"
}
```

## I adjusted the GraphQL API of a Service locally run, what now?

Assuming, you have a Spring Boot service running locally and adjusted its `.graphqls` files, maybe some business logic or JPA entities and want the changes to be propagated into the frontend (which also is set up to run locally).
First of all, make sure you didn't forgot to change a related interface entity so you don't have to repeat this, e.g. modified the entity from `content.graphqls`, but not the related one in the `mutation.graphqls` file.

1. Restart the service the change were made, so that the new changes are exposed on its `/graphql` URL path
2. Restart the gateway container to propagate the API changes to GraphQL Mesh
3. In your local frontend repo, run the `pnpm update-schema` command to pull the updated `src/schema.graphqls` file from the gateway
4. Update the queries in the frontend on the appropriate locations to utilize your GraphQL API changes
5. Run `pnpm relay` to parse the modified queries and generate fitting return types
6. Use `pnpm lint:ts` to display the resulting type errors
7. Index, commit & mention the api changes

## `docprocai` Service failing to Start

If an error occurs on the container start which contains something like:

```
could not select device driver "nvidia" with capabilities: [[gpu]]
```
```
error running hook #0: error running hook: exit status 1, stdout: , stderr: Auto-detected mode as 'legacy'nvidia-container-cli: initialization error: WSL environment detected but no adapters were found: unknown
```

The fix for this might be the following adjustment in the `doccprocai` services docker-compose file:
```diff
diff --git a/docker-compose.yml b/docker-compose.yml
index 9617824..9d54ccf 100644
--- a/docker-compose.yml
+++ b/docker-compose.yml
@@ -24,17 +24,10 @@ services:
     ports:
       - "9900:9900"
       - "9901:9901"
     depends_on:
       - database
-    deploy:
-      resources:
-        reservations:
-          devices:
-            - driver: nvidia
-              count: all
-              capabilities: [ gpu ]
   dapr-docprocai:
     image: "daprio/daprd"
     command: [
       "./daprd",
       "--app-id", "docprocai_service",
```

## Error Unsupported class file major version 64 when compiling
Try to manually force gradle to use JDK 21 in the IntelliJ settings.
