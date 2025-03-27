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

GitHub Actions is a continuous integration and continuous delivery (```CI/CD```) platform that enables automated build, test and deployment pipelines. GitHub Actions also enables the execution of non-CI/CD type automation on a repository, like adding a label when someone creates a new issue on the repository.
GitHub Actions supports Linux, Windows and macOS operating systems

## Terminology

![github-actions-overview](/images/uploads/github-actions.png)

### Event(s)

A specific activity in a repository that triggers a workflow execution
  
Examples:
  - A user creates a ```pull request```
  - A user opens an ```issue```
  - A user pushes a ```commit``` to a repository
  - Other [example events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

  {{< figure src="images/uploads/github-action-events.png" width="600" height="600">}}

{{% callout note %}}
To skip the CI workflow for a specific commit, you can include ```[skip ci]``` or ```[ci skip]``` in the commit message. GitHub recognizes this phrase and will not trigger the CI workflow for that commit.
{{% /callout %}}

### Workflow(s) 
  
{{< figure src="images/uploads/github-action-syntax.png" class="alignright">}}

A configurable automated process:

  - Made up of one or more jobs defined by a YAML associated to a repository
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

### Runner(s)

- A server that runs your workflows when they are triggered
- A given runner can execute a single job at a time
- Ubuntu Linux, Windows and macOS provided (hosted) by GitHub
- Each workflow runs on a fresh, ```newly-provisioned``` virtual machine
- Runners come in different configurations to meet specific needs
- Users can leverage GitHub hosted or self-hosted runners

### Job(s) 

A set of steps in a workflow that execute on the same runner
- Each job runs within a virtual machine (runner) OR container
- Jobs run in **parallel** by default but can be configured to depend upon one another using ```needs``` keyword
- Each job contains one or more step(s)
- Steps can be a shell command ```[run]``` or an action ```[uses]```

- Runs when:
  - Invoked within a workflow
  - When the prior / dependent job within the workflow finishes
  - Simultaneously with another job within the workflow
  - In the context of a [matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

- Examples:
  - Separate, unique build jobs for different architectures
  - A packaging job
  - A deployment job

  ```yaml
  jobs:
    job_1:
      runs-on: ubuntu-latest
      steps:
        - run: echo "The job was triggered by a ${{ github.event_name }} event."
        - run: echo "The drive is '${{ github.event.inputs.homedrive }}'."
        - run: echo "The environment is '${{ github.event.inputs.environment }}'."
        - run: echo "The log level is '${{ github.event.inputs.logLevel }}'."
        - run: echo "Should run the matrix? '${{ github.event.inputs.run_matrix }}'."

    job_2:
      runs-on: ubuntu-latest
      needs: job_1
      steps:
        - run: echo "Status ${{ job.status }}"

    job_3:
      runs-on: ubuntu-latest
      needs: job_1
      steps:
        - run: echo "Services ${{ job.services }}"

    job_4:
      runs-on: ubuntu-latest
      needs: [job_2, job_3]
      steps:
        - run: echo "Status ${{ job.status }}"
        - name: Step Summary
          run: echo "All the jobs completed in ${{ github.ref }} branch!" >> $GITHUB_STEP_SUMMARY  
  ```  

{{% callout note %}}
You can set some custom Markdown for each job summary to display and group unique content, such as test result summaries, failures and so on. ```GITHUB_STEP_SUMMARY``` is an unique environment variable for each step in a job. 
  ```sh
  echo "{markdown content}" >> $GITHUB_STEP_SUMMARY
  ```
{{% /callout %}}
  
### Step(s) 
  
Granular execution of a job
- Steps can be a shell command (```run```) or an action (```uses```)
- Data can be shared from one step to another
- When using the ```run``` command it runs in the same process/same directory in a ```shell```
- By default, GitHub Actions uses ```bash``` on Linux and ```pwsh``` on Windows. If you want to specify a different shell, you need to use the ```shell``` keyword.
- The availability of some shells may depend on the runner environment (e.g., Ubuntu, Windows, MacOS). Always ensure that the shell you are trying to use is available in your runner's environment.

- Runs when:
  - Invoked within a job
  - When the prior step finishes

- Examples:
  - A step that builds your application
  - A step that pushes an artifact to the artifact repository

  ![github-actions-steps](/images/uploads/github-action-steps.png)


### Action(s) 
  
A **reusable** step is an ```Action```

- Reduces the amount of repetitive code written in Workflow files
- Actions can be self-created / customized, or provisioned from the [GitHub Marketplace](https://github.com/marketplace?type=actions).
- Lives in a independent **public** repository.
- Can be written in JavaScript/Typescript(```NodeJS```). 
  - May use [GitHub Actions Toolkit](https://github.com/actions/toolkit) for command line argument parsing, passing parameters, interacting with the GitHub API.
  - The most minimal action definition is composed of a ```name``` and a ```runs``` object to define how the action is executed and what file to run.
  
    ```yml  
      name: "HelloWorld Action"
      description: "Says Hello!"
      author: "Avijit Chatterjee"

      runs:
        using: "node20"
        main: "dist/index.js"
      branding:
        icon: "aperture"
        color: "green"
    ```

- Could be written as a ```Docker``` container that does the the action logic and reference it from the registry like say Docker Hub when invoking the action
  ![github-actions-docker](/images/uploads/github-action-docker.png)
- Could be a ```Composite``` Action where you would basically combine multiple ```run``` and ```uses``` command steps into a single action.
  - Here's a simple example of a composite GitHub action that runs a set of commands to set up a ```Node.js``` environment, install dependencies, and run tests.
    ```yml
    name: 'Setup Node.js'
    description: 'Sets up a Node.js environment, installs dependencies, and runs tests'
    inputs:
      node-version:
        description: 'Node.js version to use'
        required: true
        default: '14'
      working-directory:
        description: 'The directory to run commands in'
        required: false
        default: '.'
    outputs:
      test-results:
        description: 'The result of the test run'
        
    runs:
      using: "composite"
      steps:
        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: ${{ inputs.node-version }}

        - name: Install Dependencies
          run: npm install
          working-directory: ${{ inputs.working-directory }}

        - name: Run Tests
          run: npm test
          working-directory: ${{ inputs.working-directory }}
          continue-on-error: true
          id: tests

        - name: Set Output for Tests
          run: echo "test-results=${{ steps.tests.outcome }}" >> $GITHUB_ENV
    ```
- Whichever way you create it the ```Action``` definition is stored in the ```action.yml``` file, put directly in the root of the repository.

- Use ```Semantic``` versioning to build and maintain your Actions. When you or others refer/use your Action you would do so like: 
  - {owner}/{repo}@{ref}
  - {owner}/{repo}/{path}@{ref}
  - where ref could be SHA / Tag / Branch
  ![github-actions-reference](/images/uploads/github-action-reference.png)

- Pass variables to Action

  - **with:**
    The ```with``` keyword allows you to specify inputs for actions. These inputs can be defined in the action.yml file and can have default values, data types, and descriptions. 
    This method is typically used for passing configurable parameters that the action will use.

  - **env:**
    The ```env``` keyword is used to set environment variables that can be accessed in the action. Environment variables are global within the job and can be useful for passing 
    sensitive data (e.g., secrets) or shared configurations.

- Examples:
  - Pull a given git repository for GitHub
  - Set up the correct toolchain for a build environment
  - Establish authentication to a cloud provider

### Context

Every workflow execution has access to different ```Context``` information during runtime. Contexts are a way to access information about workflow runs, variables, runner environments, jobs, and steps. Each ```Context``` is an object that contains properties, which can be strings or other objects.

You can access contexts using the expression syntax: ``` ${{ <context> }} ```. 
We have already seen some usage like this where we output the branch name from the ```github``` context object

```sh
  echo "All the jobs completed in ${{ github.ref }} branch!" >> $GITHUB_STEP_SUMMARY
```

We can look at all the different ```context``` objects available in a workflow and we can do this by converting the objects to JSON and storing them in an environment variable on each step. Lastly, we'll output that environment variable with a simple echo statement:

- Create a new workflow with the below code: ```.github/workflows/context.yaml``` . 

  ```yaml
  name: Context Information
  on:
    push:
      branches-ignore: main
  jobs:
    show-context:
      name: Show Context
      timeout-minutes: 5
      runs-on: ubuntu-latest-internal
      steps:
        - name: Dump Event Information
          env:
            CONTEXT_ITEM: ${{ toJson(github) }}
          run: echo "GitHub context ${CONTEXT_ITEM}"
        - name: Dump Job Information
          env:
            CONTEXT_ITEM: ${{ toJson(job) }}
          run: echo "job context ${CONTEXT_ITEM}"
        - name: Dump Runner Information
          env:
            CONTEXT_ITEM: ${{ toJson(runner) }}
          run: echo "runner context ${CONTEXT_ITEM}"
        - name: Dump Step Information
          env:
            CONTEXT_ITEM: ${{ toJson(steps) }}
          run: echo "step context ${CONTEXT_ITEM}"
  ```
- ```Add``` & ```Commit``` your changes, then ```Push``` your branch.
- Go to your repository and view the ```Actions``` tab to see the execution against your published branch.
- The result will be an execution of the workflow whenever any changes are pushed **EXCEPT** on the ```main``` branch. ```Context``` information is viewable in the output so that you can understand how to utilize those values in your workflows
  ![github-actions-context](/images/uploads/github-actions-context.png)

{{% callout question %}}
 Why is the ```steps``` context empty?
{{% /callout %}}

{{% callout note %}}
 
In GitHub Actions, the steps context is only populated with information about the steps that have already run in the current job and have an ```id``` specified. We will apply the necessary property ```id``` so we can pass data between steps. 

```yaml
name: Context Information
on:
  push:
    branches-ignore: main
jobs:
  show-context:
    name: Show Context
    timeout-minutes: 5
    runs-on: ubuntu-latest-internal
    steps:
      - name: Provide Some Step Outputs
        id: step-outputs
        run: echo "TRUE_STATEMENT=Possums are terrible." >> $GITHUB_OUTPUT
      - name: Dump Step Information
        env:
          CONTEXT_ITEM: ${{ toJson(steps) }}
        run: echo "${CONTEXT_ITEM}"
```
The result will be that the steps context now has the data from the previous step.
![github-actions-steps-context-output1](/images/uploads/github-actions-steps-context-output1.png)
It's possible to pass multiple values in a single step. This will teach you how to accomplish that and then access both values.

```yaml
name: Context Information
on:
  push:
    branches-ignore: main
jobs:
  show-context:
    name: Show Context
    timeout-minutes: 5
    runs-on: ubuntu-latest-internal
    steps:
      - name: Provide Some Step Outputs
        id: step-outputs
        run: |
          echo "TRUE_STATEMENT=Possums are terrible." >> $GITHUB_OUTPUT
          echo "FALSE_STATEMENT=Possums are great." >> $GITHUB_OUTPUT
      - name: Dump Step Information
        env:
          CONTEXT_ITEM: ${{ toJson(steps) }}
        run: echo "${CONTEXT_ITEM}"
```

The result will be that the steps context now has even more data than the previous step.
 ![github-actions-steps-context-output2](/images/uploads/github-actions-steps-context-output2.png)
You can see it's just a ```JSON``` object and step ```id``` is the **key**, that is being put in the JSON object. It has to have something or else every single step that was logged here would look exactly the same and would be ambugious. 
That is why GitHub Actions **enforce** that if you don't include an  ```id```, it would not log anything out.

{{% /callout %}}

## Further Read

That's all for now.  Consider *Intro to GitHub Actions* complete, but there's a long ways to go and lots more to learn. 

[Understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)