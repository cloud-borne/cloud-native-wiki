---
title: Actions
linktitle: ⚙️ Actions
toc: true
type: book
date: "2019-05-05T00:00:00+01:00"
tags:
  - GitHub

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 5
---

GitHub Actions Cheat Sheet

<!--more-->

## Overview

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that enables automated build, test and deployment pipelines. GitHub Actions also enables the execution of non-CI/CD type automation on a repository, like adding a label when someone creates a new issue on the repository.
GitHub Actions supports Linux, Windows and macOS operating systems

## Terminology

![github-actions-overview](/images/uploads/github-actions.png)


**Event(s)** - A specific activity in a repository that triggers a workflow execution
- Examples:
  - A user creates a pull request
  - A user opens an issue
  - A user pushes a commit to a repository
  - Other [example events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

**Workflow(s)** - A configurable automated process
  - Made up of one or more jobs
    - Defined by a YAML associated to a repository
      - Found in `.github/workflows`
      - A given repo can have multiple workflows
    - Runs when:
      - Triggered by an event in a repository
      - Triggered manually
      - Triggered via another workflow (workflows can reference other workflows)
      - Posting to a REST API
      - Per a defined schedule
      
      Examples:
      - A workflow to build and test pull requests
      - A workflow to deploy your application every time a release is created
      - A workflow to add a label every time someone opens a new issue

**Runner(s)** - A server that runs your workflows when they are triggered
- A given runner can execute a single job at a time
- Ubuntu Linux, Windows and macOS provided (hosted) by GitHub
- Each workflow runs on a fresh, newly-[provisioned virtual machine
- Runners come in different configurations to meet specific needs
- Users can leverage GitHub hosted or self-hosted runners

**Job(s)** - A set of steps in a workflow that execute on the same runner
- Each job runs within a virtual machine (runner) OR container
- Jobs can be configured to depend upon one another
- Each job contains one or more step(s)
- Runs when:
  - Invoked within a workflow
  - When the prior / dependent job within the workflow finishes
  - Simultaneously with another job within the workflow
  - In the context of a [matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
  
  Examples:
  - Separate, unique build jobs for different architectures
  - A packaging job
  - A deployment job

**Step(s)** - Granular execution of a job
- Steps can be either a shell script or an Action
- Data can be shared from one step to another
- Runs when:
  - Invoked within a job
  - When the prior step finishes
  Examples:
  - A step that builds your application
  - A step that pushes an artifact to the artifact repository

**Action(s)** - A custom application for the GitHub Actions platform that performs a complex but frequently repeated task
- Reduces the amount of repetitive code written in Workflow files
- Actions can be self-created / customized, or provisioned from the GitHub Marketplace.
  Examples:
  - Pull a given git repository for GitHub
  - Set up the correct toolchain for a build environment
  - Establish authentication to a cloud provider

That's all for now.  Consider "Intro to GitHub Actions" complete, but there's a long ways to go and lots more to learn. 

## Further Read

[Understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)