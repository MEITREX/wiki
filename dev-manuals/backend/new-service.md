# Adding a new service

To add a new service create a new repository from the template-microservice using the Use this template button. 
The name should be \<servicename\>_service and the visibility should be public.
Once the repository has been created go to the repository settings and add the "Feature Dev" team to the collaborators. 
Make the main branch protected with the following settings:
![](/images/branch-protection-rules.png)

Name the branch protection rule "MEITREX-repo-rules".

Change the License to MIT, if it isn't MIT yet.

The following files need to be changed:

- Settings.gradle -> change the "rootProject.name" to the servicename
- Build.gradle -> under sonarqube change the property "sonar.projectKey" to "MEITREX_\<servicename\>"

- Change the name of the template package to the name of the service
- Remove the package-info.java files in the src/main/java folder, if the file is present (or update with the microservice specific information)
- Update the application.properties file in the src/main/resources folder (check the TODOS in the file)
- Update the application-prod.properties in the src/main/resources folder (check the TODOS in the file)
- Update the application-dev.properties in the src/main/resources folder (check the TODOS in the file)
- Update the application.properties in the src/test/resources folder (check the TODOS in the file)
- Update the docker-compose.yml 

- Define the GraphQL schema in the src/main/resources/schema.graphqls file

After the service has been created and updated you need to do the following:

- Add the repository to sonarcloud. You need admin permissions in sonarcloud to successfully complete this part
  - Follow the instructions for extra configuration. Click Configure analysis in cour CI ![](/images/sonarcloud-instructions-1.png)
  - Unselect automatic analysis
  - choose GitHub actions, only the first step needs to be completed
- Add SONAR_TOKEN to the secrets on GitHub

- Add the new Ports to the wiki architecture ports.
