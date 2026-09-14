# ISEC6000 Assessment 2: Jenkins Setup

Name: Pabin Gurung
Student ID: 22343460

## About this project

This repository contains the Docker configuration for my Jenkins environment.

Jenkins checks the application repository for changes, runs a unit test,
checks dependencies for security issues, and builds and uploads a Docker
image when the checks pass.

The application code and Jenkinsfile are stored in a separate repository.

## Project links

Application repository:
https://github.com/GurungPabin/aws-elastic-beanstalk-express-js-sample

Jenkins configuration repository:
https://github.com/GurungPabin/isec6000-assessment2-jenkins

Docker Hub repository:
https://hub.docker.com/r/pabingurung22343460/isec6000-assessment2

## Main files

- compose.yaml: Sets up the three containers, their connections and storage.
- jenkins/Dockerfile: Installs Jenkins and the required plugins.
- agent/Dockerfile: Prepares the worker that runs the pipeline.
- .env: Stores the private agent connection secret locally. It is not uploaded to GitHub.

## How the setup works

There are three services:

1. Jenkins manages the pipeline and keeps its results.
2. The agent runs the pipeline tasks.
3. Docker-in-Docker provides a separate Docker engine for building images.

The agent is named docker-agent. Jenkins itself has no build executors,
so pipeline tasks are assigned to the agent.

The pipeline uses Node 16.20.2 for installing dependencies, running the
unit test and checking dependencies.

## Security settings

- Jenkins is accessed through an SSH tunnel.
- Its web port is available only on the server's local address.
- Users cannot register their own accounts.
- A separate builder account runs the pipeline with limited permissions.
- The builder cannot administer Jenkins or change job settings.
- Jenkins stores the Docker Hub token as a credential.
- Connections to the Docker engine use certificates and encryption.
- Jenkins and the agent run as non-root Linux users.

Docker-in-Docker requires privileged mode. This remains a security
limitation, so this environment should only run trusted pipeline code.

## Saved data

Docker volumes keep the Jenkins settings, accounts, build logs and reports.
Other volumes keep Docker images, certificates and agent workspaces.

This allows the containers to restart without losing the saved data.

## Starting the existing setup

The server needs Docker and Docker Compose. The local .env file must
contain the agent connection secret.

From this repository folder, run:

    sudo docker compose config --quiet
    sudo docker compose up -d --build
    sudo docker compose ps

These commands start an environment whose Jenkins accounts, agent,
credentials and pipeline job have already been configured.
A new installation needs those settings to be completed first.

Keep the SSH tunnel open and visit http://localhost:8080 on your computer.

To stop the services while keeping their saved data:

    sudo docker compose stop

Do not use docker compose down -v when you want to keep the saved data.

## Pipeline settings

Job name: 22343460_Assessment2_pipeline

Jenkins reads the Jenkinsfile from the application repository's main branch.
Poll SCM checks for changes every five minutes.

The pipeline runs as pabin-builder and uses the Docker Hub credential
named dockerhub-credentials.

It performs these steps:

1. Downloads the application code.
2. Installs dependencies.
3. Runs the unit test.
4. Checks dependencies for vulnerabilities.
5. Builds the application image.
6. Uploads the image to Docker Hub.

A High or Critical dependency finding stops the pipeline before the image
is built or uploaded.

Test and security reports are saved with each build.
The pipeline is configured to retain the latest 20 builds.

## Verification results

- Build 3 started automatically after a GitHub change and passed.
- Build 4 tested the security check using a temporary vulnerable dependency.
  It detected a High finding and skipped the image build and upload.
- The temporary dependency was removed.
- Build 5 passed and uploaded Docker image tag 5.
- Tag 4 was not published.
- Build 5 reported 3 Moderate findings and no High or Critical findings.

Passing the security check does not mean the application has no
vulnerabilities. This check covers application dependencies, not the
operating system packages inside the Docker image.

## Problems resolved

The builder initially lacked permission to run on the agent.
Granting Agent/Build permission allowed the pipeline to run.

The pipeline initially could not access the Docker Hub credential.
Enabling the Credentials/UseItem permission and granting it to the
builder resolved the issue.

An indentation error in compose.yaml was corrected.
The file was checked before restarting Jenkins.
