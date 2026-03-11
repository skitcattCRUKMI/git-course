---
title: "Continuous Integration and Continuous Deployment (CI/CD)"
teaching: 20
exercises: 10
questions:
  - "What is CI/CD?"
  - "Why is CI/CD useful?"
  - "What are the key parts of a CI/CD pipeline?"
  - "How does GitLab use a .gitlab-ci.yml file?"
  - "What are some good CI/CD practices?"
objectives:
  - "Explain the purpose of CI/CD."
  - "Describe the main parts of a GitLab CI/CD pipeline."
  - "Read and write a simple .gitlab-ci.yml file."
keypoints:
  - "CI/CD automates building, testing, and deploying software."
  - "Continuous Integration checks changes frequently and automatically."
  - "Continuous Delivery keeps software ready to release."
  - "Continuous Deployment automatically releases changes after checks pass."
  - "A .gitlab-ci.yml file defines stages, jobs, and rules for a pipeline."
  - "Automated tests provide quick feedback and reduce integration problems."
---

## Introduction to CI/CD

Continuous Integration (CI) and Continuous Delivery/Deployment (CD) are practices that automate key parts of software development. In GitLab, this automation is defined in a file called `.gitlab-ci.yml`.

When changes are pushed to the repository, GitLab can automatically create a pipeline. The jobs in that pipeline are then picked up by available runners. In our setup, those jobs are executed by runners on our GitLab Runner server.

CI/CD helps teams work more reliably by automating routine checks and reducing manual steps.

- **Continuous Integration (CI)** means developers merge changes frequently, and automated checks run on those changes.
- **Continuous Delivery/Deployment (CD)** means software is kept ready to release, or is released automatically after checks pass.
- **Runners** are the agents that execute pipeline jobs such as build, test, and deploy tasks.

This means that a simple `git push` can trigger a chain of automated steps, helping teams detect problems early and get faster feedback on their changes.

## Why is it useful?

CI/CD can help teams to:

- find problems early
- reduce manual errors
- get faster feedback
- improve collaboration
- release changes more safely and more often

## Main parts of a CI/CD pipeline

A CI/CD pipeline is made up of a series of automated steps. These steps usually run in a fixed order, so that problems are caught early before later stages begin. A simple pipeline often starts by preparing the software, then checking it, and finally releasing it.

Common stages in a pipeline include:

- **Build**  
  The project is compiled, packaged, or otherwise prepared for testing or release.

- **Test**  
  Automated tests check that the software behaves as expected. These may include unit tests, integration tests, or other validation steps.

- **Deploy**  
  If earlier stages succeed, the software can be deployed to a staging or production environment.

- **Feedback**  
  Pipeline logs, reports, and job results help developers understand what happened and identify any failures quickly.

Not every project uses exactly the same stages, but the general idea is the same:

- prepare the software
- check that it works
- release it safely
- review the results

## The `.gitlab-ci.yml` file

GitLab uses a `.gitlab-ci.yml` file to define the pipeline.

Common parts of the file include:

- **stages**: the order of work
- **jobs**: the tasks to run
- **script**: the commands in each job
- **image**: the environment used to run jobs
- **rules**: when jobs should run
- **artifacts**: files saved after a job finishes
- **variables**: values shared across jobs

A fuller breakdown of how to structure this file can be found [here](https://docs.gitlab.com/development/cicd/templates/) 

## Example `.gitlab-ci.yml`

```yaml
  variables:
    GIT_STRATEGY: clone

  stages:
    - BUILDANDPUSH
    - DEPLOY

  build-image:
    timeout: 2h
    stage: BUILDANDPUSH
    tags: 
        - gitlabrunner_build
    script:
        - docker build -t sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:dev .
        - docker push sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:dev
        - docker rmi sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:dev
        - docker builder prune -f
    only:
      refs:
        - dev

  deploy-pull-image:
    stage: DEPLOY
    tags:
        - gitlabrunner_build 
    script:
        - docker pull sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:dev
        - docker tag sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:dev sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:latest
        - docker push sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:latest
        - docker rmi sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:dev
        - docker rmi sc-docker-registry.scicom.picr.man.ac.uk/shiny_app:latest
        - docker builder prune -f
    only:
      refs:
        - main

```

## Reading the example

This example shows some of the main components used in a GitLab CI/CD configuration file. Each part controls a different aspect of how the pipeline runs.

- **`variables`**  
  Variables define values that can be reused throughout the file. In this example, `GIT_STRATEGY: clone` controls how the repository is fetched before a job runs.

- **`stages`**  
  Stages define the order in which groups of jobs are run. Here, the stages are `BUILDANDPUSH`, `DEPPLOY_TEST`, and `DEPLOY`.

Each job is given a name and assigned to one of these stages.

- **Job names**  
  `build-image` and `deploy-pull-image` are the names of two jobs in the pipeline.

- **`stage`**  
  The `stage` keyword tells GitLab which stage a job belongs to.

Jobs also contain the commands that should be executed by the runner.

- **`script`**  
  The `script` section lists the shell commands run for that job.

Some jobs include extra settings to control where and how they run.

- **`tags`**  
  Tags are used to match jobs to specific runners. A job with the tag `gitlabrunner_build` will only run on a runner with the same tag.

- **`timeout`**  
  The `timeout` keyword sets the maximum amount of time a job is allowed to run.

The file also controls when particular jobs should be included in a pipeline.

- **`only: refs`**  
  This limits a job to specific branches or refs. In this example, one job is limited to `dev` and the other to `main`.

## Conclusion

CI/CD improves software development by automating routine checks and deployments. In GitLab, the `.gitlab-ci.yml` file describes how the pipeline works. A simple pipeline with build, test, and deploy stages can provide fast feedback and improve software quality.

{% include links.md %}