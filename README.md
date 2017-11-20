Petclinic deployment process
============================

This repo contains code for the automated deployment process of a basic grails app. 
It contains the application code and an ansible playbook which is used to deploy the war into a tomcat7 servlet container.
The application code was forked from https://github.com/secretescapes/grails-petclinic.
This should be run from circleCI and contains a config.yml file under the .circleci directory.

## CircleCI ##

To run this you need to sign up to circle CI and start a project with this repo. Please fork the repo as you will need write access to the repo to kick off a build.
The job will run on every commit to the repo. Currently there is a hardcoded version in the config.yml which should match the application version and will be used to tag the artifacts.
Past jobs can be rerun at any time to deploy an older version of the code.


## Variables ##

There is a file called hosts which has a list of ips to run this playbook against and the user to ssh onto the server as currently this is just the private ip of the vagrant image provided.
There are 2 command line parameters deploy_version and path_to_war. This allows the deploy script to be run locally as well as on the CI server. 
If they are not provided the defaults in deploy.yml will be used.


## Access Control ##

The VM provided had a ssh password however using this is not recommended as it is less secure than using ssh keys.

ssh keys can be stored on CircleCI https://circleci.com/docs/1.0/permissions-and-access-during-deployment/. 
This means users do not need server access to be able to trigger a deployment. Only people with write access to the repo can trigger a build so deployment is restricted to authorized users only.

The path to the private key of the server to deploy to can be specified on the command line with the parameter --private-key 


## Rollback ##

To deploy a previous version of the application code you need to navigate to the workflow section of CircleCI. 
From there you can rerun any of the previous deploy jobs to redeploy an older version of the code.


## Running it locally ##

This deployment can be run locally by users who have the server ssh keys stored locally.
If this is being run against a vagrant VM the key path can be found be running the "vagrant ssh-config" command.
Firstly you need to build the war ./gradlew war. The path to this war artifact (including the filename) should be used in the deploy command.

Then you can run the ansible command 

```
ansible-playbook -i hosts deploy.yml --private-key $PLEASE_REPLACE_ME --extra-vars "deploy_version=$PLEASE_REPLACE_ME path_to_war=$PLEASE_REPLACE_ME"
```

## Improvements ##

The focus of this task was on creating a minimum viable solution, therefore there were considerations that were left out, some of which are mentioned below.

Repository organisation - currently both the application code and ansible playbook are stored in the same repository to simplify things as this was an MVP solution.  
In a production solution it would be better to separate the deployment code from the application code.

Version - currently the deploy version is hardcoded into the circleCI config file and has to be specified multiple times which leaves room for human error. 
It would be better if the version could be generated dynamically from the git hash.

Artifactory storage - currently the artifacts are just passed between the build and deploy steps on CircleCI however it would be better to have them backed up somewhere

Healthchecks - It would be good to have at least one healthcheck running on the deployed server to verify if the deployment was a success or not.

Logging and observability - for a production system you would want to build in logging standards and metrics collection from the start.

Uptime deployments - this was not a requirement and so hasn't been addressed in the solution but it could be looked at

Cleanup - the solution does not clean up old artifacts which would become a problem with time

Testing - if the application code and deployment code were in separate repos a set of tests could be added to the deployment code so that changes to the ansible playbook could be made with confidence.


