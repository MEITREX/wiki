# Deploy a new Service

**Prerequisites**:

- Cluster Access
- Actual tfstate file
- Cloned infrastructure repository

To deploy a new service, we need a Terraform file which describes the new service. We provide a template_service.tf file to simplify deployment. We will go through that template with an example service.

To start, make a copy of the template_service.tf file and rename it with your service name, e.g. example_service.

Open the file in a text editor of your choice. 

First, delete the first `\*` and last line `*\` of the file to uncomment the resources. 

## Structure 

The template service consists of a

- **Deployment resource**: The deployment controls the pod in which your container and the Dapr sidecar is hosted 
- **Helm release of postgres**: The helm resource deploys a PostgreSQL database used by your service
- **Password resource**: Generates a password used for database authentication and stores it as a Kubernetes secret. 

To deploy the service, we need to set the service names. The following is a list of the places where you need to make adjustments in order to run the service.

At each location, there is a placeholder. 

**<template\>_service** replace it with your service name. 

For example **<template\>_service** will replaced by **example_service**.



## Deployment resource

In the deployment resource, you need to set the name of the service in the following parts. 

- The name of the resource  ```resource "kubernetes_deployment" "gits_<template>_service" ```
- The helm_release name in the depends on variable `depends_on = [helm_release.<template>_service_db, helm_release.dapr, helm_release.keel]`
- The name attribute in the metadata section ``` metadata {
 name = "gits-<template>-service"```
- The app attribute in the labels section `labels = {
 app = "gits-<template>-service"
 }`
- The app attribute in the selector section `selector {
 match_labels = {
 app = "gits-<template>-service"
 }
 }`
- The app variable in the template resource `template {
 metadata {
 labels = {
 app = "gits-<template>-service"
 }`
- The dapr app id in the annotations `"dapr.io/app-id" = "<template>-service"`
- The name in the container section `name = "gits-<template>-service"`
- You need to set the service name also in the spring datasource url to create the database with the same name as the service. `env {
 name  = "SPRING_DATASOURCE_URL"
 value = "jdbc:postgresql://<template>-service-db-postgresql:5432/<template>-service"
 }`
- Also set the name of the password resource `env {
 name  = "SPRING_DATASOURCE_PASSWORD"
 value = random_password.<template>_service_db_pass.result
 }`
- Lastly, set the image name in the container spec sections `image  = "ghcr.io/meitrex/<template>_service:latest"`

**Note**: If there is no image in the packages tab of the meitrex organisation. Check the build Docker image GitHub Actions pipeline. It is possible that the image name is not correctly set there.

You also need to set the app and HTTP ports. 
- in the annotations `"dapr.io/app-port" = **01` and `"dapr.io/http-port" = **00`
- And the ports in the liveness and readiness probes. Usually, we use the dapr app port `**01`. 

## Helm release resource

- First, set the name of the resource correctly `resource "helm_release" "<template>_service_db"`
- The name of the db `name = "<template>-service-db"`
- The auth database variable of the postgres helm chart `set {
 name  = "global.postgresql.auth.database"
 value = "<template>-service"
 }`
- The password environment variable of  the postgres database `set {
 name  = "global.postgresql.auth.password"
 value = random_password.<template>_service_db_pass.result
 }`

## Password resource

Here you simply need to change the name of the password resource `resource "random_password" "<template>_service_db_pass"`
  
## Environmental Variables

If your services use environment variables, you need to set them in the Terraform script as well. For that, you can use the env Block in the template section. `env {
 name  = "EXAMPLE_VALUE"
 value = example
 }`.
If your service is developed with Spring Boot, you can simply use the declared variable in the properties file. For example: **example.value** replace the dot `.` with an underscore `_` and write the name in capital letters. Example Spring Boot will automatically set the variable.  

If your service uses another framework, make sure that the service reads the environment variable. 

## Health, Liveness & Readiness probe

The health, liveness and readiness probe is used by Kubernetes to check if the service booted up correctly and is ready to serve requests. 

Spring Boot provides these endpoints, and they are enabled by default when you use the template microservice repository. Otherwise, enable them. 

If your service uses other frameworks, make sure that it provides the endpoints and enable them. If this is not possible, you can also delete the probes from the deployment file; otherwise, your service will not run correctly on the cluster. 

**Note** If your service takes longer than 60 seconds to boot up, adjust the  `initial_delay_seconds` variable accordingly. This will increase or decrease the wait time before Kubernetes checks if the service is booted up correctly. Otherwise, it will assume that the service is not booted up correctly and terminate it, which will cause a crash back in a loop.  