---
layout: post
title:  "GitHub Actions & Java - Part 1"
date:   2021-01-30 00:00:00 +0
categories: java
---

This is the first of a series of articles where I'm relating my experience with GitHub Actions, building and deploying Java microservices to production. In each article, I'm focusing on the key aspects of the framework and how we can take advantage of its features in the different phases that a Java CI/CD pipeline expects. The rest, you can easily find out in the [official GitHub Actions documentation](https://docs.github.com/en/actions).

### Why GitHub Actions?

As a Java developer coding and delivering software for a few years, I consider that working on a Continuous Integration and Deployment \(CI/CD\) environment is one of the most pleasant experiences in day-to-day work. With the right automation, repetitive and erroneous manual tasks can be avoided and everyone is able to focus more on what's important. The workspace becomes much more efficient, clever and an exciting place to be.

Developing and maintaining all this magic is not usually at first. I've worked with a few CI/CD systems such as Jenkins, Bamboo \(and more recently with GitLab\), and I always felt a big gap between the required core skills for our day-to-day tasks and the ones to deal with these systems. The industry that cultivated for so long the _DevOps_ concept, quickly evolved to a _NoOps_ standard that apparently became to stay. 

I always enjoyed to build CI/CD pipelines - playing with Jenkins back and forward, configuring steps in jobs, that called other jobs, and with some shell script and tools/plugins on top, something supposed to happen at the end, usually happened. A few months ago, I had a job in hand to set up a CI pipeline for a few projects where Jenkins was not an option. These projects were hosted on GitHub and the first thing that came to my mind was to use Travis or CircleCI \(from my experience in open-source projects\). Someone told that we could try GitHub Actions. After reading the docs and giving a try, I just realized  - Shit, finally a decent _NoOps_ solution for every developer. If the same is happening to you, have a look on this article from GitHub on how to [Migrate from Jenkins to GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/migrating-from-jenkins-to-github-actions).

In the next sections I'll explain how to build a Java project that has a JAR artifact as an outcome.

### Setup & Build

First things first. GitHub Actions is a decentralized system - this means that in opposition to systems like Jenkins that all the configuration is in a single place, with GitHub Actions all the configuration will be hosted in the project repository. This doesn't sound something new, but I wouldn't expect anything else nowadays where cross-functional teams own projects and are responsible for the full development life-cycle.

 In GitHub Actions, regular CI _jobs_ are called `workflows` that are triggered by `events` and contain `jobs` . A job can define multiple `steps` that run **Actions**. Simple, right? For more information on how to get started, check the [official documentation](https://docs.github.com/en/actions/quickstart).

To demonstrate some of the functionalities of GitHub Actions I just developed a [nonsense Java microservice](https://github.com/rsaestrela/vertx-java-ms) with [Vert.x](https://vertx.io/) \(it could have been with Spring or anything else\).  The only important points for now is that this service is built with Maven, uses a PostgreSQL database and runs Flyway migrations.

#### Workflow

This is the workflow created to build the project when there's a push branch `main`. Additionally, GitHub Actions allows us to define different ways to trigger workflows. One of them is `workflow_dispatch`. This item was left empty just to allow triggering the job [manually](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) without any parameter. There are multiple ways of triggering workflows being the `workflow_dispatch`or `repository_dispatch` \(not included in this article\) probably the most complex but interesting ones. For more information on this topic check the [official documentation](https://docs.github.com/en/actions/reference/events-that-trigger-workflows).

```yaml
name: build-main
on:
  push:
    branches:
      - main
    workflow_dispatch:
jobs:
  build:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres
        env:
          POSTGRES_DB: vertxbroker
          POSTGRES_USER: admin
          POSTGRES_PASSWORD: admin
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5455:5432
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: 11
          java-package: jdk
          architecture: x64
      - id: build
        run: mvn verify
```

#### Setup Dependencies

As mentioned before, workflows are simply containers of jobs. Jobs are virtually executed in an OS hosted in a... container. A job must define a [runs-on](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions?query=runs-on#jobsjob_idruns-on) configuration that specifies the OS used in this instance and may specify a list of [service containers](https://docs.github.com/es/actions/guides/about-service-containers) - services that will be installed in the system. In this project, a `postgres` service was defined to spin a PostgreSQL database necessary to build the project \(and eventually execute integration tests using it\).  There are a few different types of service containers [available](https://github.com/actions/example-services) for these purposes.

#### Just build it!

With all the dependencies set, only 3 `steps` must be defined within this job to build the project. This is where things get interesting. Actions aren't just triggers developed by GitHub. Everyone in GitHub \(including yourself\) can build actions, which means that you can find an action for almost everything \([just keep it simple](https://youtu.be/gwZ81gJRSuQ?t=72)\). A GitHub Action is specified to be executed in each step:

* actions/checkout - gets the the Git repository
* actions/setup-java - installs JDK 11 for x64 and a few Java based tools, like Maven
* custom action **build** - runs whatever we want. In this case, it's just triggering a Maven execution.

And that's it for now. After running `maven verify` a JAR binary should be built and made available in the _target_ folder. 

Without having to set up any CI tool or to master any GUI / complex DevOps configuration setup, GitHub gives _ootb_ all this functionality in an extensible and free to use framework that in my opinion \(and thanks to GitHub popularity\) will set a new standard in the industry and it will reinforce even more the brand. I don't think that Jenkins or any other powerful  CI/CD platform will be abandoned sooner, but if the Git product I'm adopting already gives me this functionality, why should I care about adding more tools and complexity to an CI/CD environment that should be easy to understand and maintain? At this stage, this might sound an overrated opinion about GitHub Actions - but believe me, as I'll show you in the next articles or this series, GitHub seems able to cover almost all the functionalities needed in a CI/CD system, at least for the most common use cases.

All the source code presented can be found in this [GitHub repository](https://github.com/rsaestrela/vertx-java-ms).