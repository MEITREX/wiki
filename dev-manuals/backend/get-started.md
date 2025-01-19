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

## Debugging/ Running Selected Services Locally

To facilitate easy debugging of services, the docker containers are set up to expose all important ports to the host machine. This can easily be seen in Docker Desktop. Port mappings can also be
found [on the wiki](https://meitrex.readthedocs.io/en/latest/dev-manuals/backend/Ports.html).

This makes it possible for you to start a service in your favorite IDE (where you can set breakpoints etc.) and for that service to hook into the rest of the system.

* At first, stop the MEITREX docker container and remove the set of services you want to run locally from the `compose.bat`/`compose.sh` list of paths to the submodule's `docker-compose.yml`s

If the service to be run locally is the frontend:

* Start the docker containers again (as described above) and follow the instructions provided in the frontend repos `README.md`

Else:

* Navigate to the `graphql_gateway` submodule, open its `docker-compose.yml` and adjust the URLs of the set of services you want to run locally in the `environment` section as follows: http://~~app-\<service-name\>~~host.docker.internal:\<service-port\>/graphql
* Rebuild & restart the docker services via `compose.bat`/`.sh up --build`. The `gateway` container should now restart frequently because it can't reach the service to be run locally. This is desired behavior, since it now tries to fetch the adjusted URLs services graphql api from the machines `localhost`.
* Make sure the Spring Bott dev profile's database port is set correctly
* Start the set of service you want to run locally on your machine using the dev profile, which can either be selected in the IDEA Ultimate  "Run Configurations", the `application.properties` file or be passed to gradle as command line argument: `gradle bootRun --args="--spring.profiles.active=dev"`
<!-- TODO python? -->
* The Dapr PubSub Events do not work when debugging in this way

# Some Hints/Common Issues

## Error Unsupported class file major version 64 when compiling
Try to manually force gradle to use JDK 21 in the IntelliJ settings.
