---
layout: post
title:  "Buildpacks with Paketo"
date:   2020-05-15 00:00:00 +0
categories: java
---

Containerization of applications has been a massive trend in the IT Industry. They provide a logical packaging mechanism in which applications can be abstracted from the environment they run, allowing them to be deployed consistently. The separation of concerns they provide, makes the developer's life easier as they can focus more in the application and product development.

In opposition to visualizing a complete hardware stack, tools like Docker and Kubernetes quickly established a new standard in the Industry to develop, pack and deploy applications. IT operations teams are now more productive and focused in delivery, building future proof solutions that are flexible and which integration is possible among different cloud service providers such as Google Cloud and Amazon Web Services.

Organizations should be aware of this. As most of these tools are free and open source - running applications in containers can be substantially cheaper than traditional setups and they have major importance when it comes to attract talent.

## Developer

NoOps is a reality. The effort IT operations needed to keep software running is being minimized and they are now more than ever focused in the improvement of processes and all the operational overhead these new setups require. For Developers, the transition to this new reality can be cumbersome. In more complex environments, tools like Docker and Kubernetes can be extremely hard to master and in some situations can simply be an overkill. In the last few years, organizations have been working in new tools no minimize the cognitive burden, improve consistency and deliver applications faster. One of this tools is **Buildpacks**.

### Containerization with Docker

Containerization of applications using Docker showed a strong appetite to popularize Docker images based on **Dockerfiles**. These Docker images, created by Organizations and Individuals have been helping us to quickly containerize applications and setup their dependencies.

As a developer, I must say that there should be a Docker image available in Internet for most of our purposes. Although, this practice lead the industry to create different solutions, that are hard to maintain and the lead it comes to inconsistencies on upstream dependencies and performance issues - when it comes to Java applications this approach has a major [performance impact](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1).

### Cloud Native Buildpacks

Cloud Native Buildpacks are part of the answer to this problem. The concept of _Buildpack_ was first conceived by Heroku is 2011, since then they have been adopted by Cloud Foundry and and other PaaS. However the need to create a standard took Pivotal and Heroku to start the Cloud Native Buildpacks project in the beginning of 2018.

> The project aims to unify the buildpack ecosystems with a platform-to-buildpack contract that is well-defined and that incorporates learnings from maintaining production-grade buildpacks for years at both Pivotal and Heroku. Cloud Native Buildpacks embrace modern container standards, such as the OCI image format. They take advantage of the latest capabilities of these standards, such as cross-repository blob mounting and image layer "rebasing" on Docker API v2 registries.

#### Paketo Buildpacks

Paketo are Modular Buildpacks written in Go that leverage and contribute to the Cloud Native Buildpacks framework. They provide different packaging flavors for different kind of systems, written using different programming languages, for different purposes.

### Lets Pack, Push and Deploy!

Lets put all this in practice. Consider building a Spring Boot Maven project - we will pack the application, push the container image to Google Cloud Repository, and deploy it with Google Kubernetes Engine.

> Intermediate steps such as tools installation, project creation and authentication won't be described. The detailed information can be found in every corner of the Internet.

#### Pack

The process of packing \(or packaging\) involves the usage of the _pack CLI_. It will run a process upon Docker that automatically discovers the dependencies required for the application to be built. In the root folder of your application project run:

`pack build gcr.io/{GC_APP_ID}/app --builder cloudfoundry/cnb:bionic`

> A `builder` must be specified - it bundles all the information how to build the app, such as buildpacks and build-time image, as well as executes the buildpacks against the application source code.

It will detect and fetch all the buildpacks needed and will orchestrate their execution. Note the packeto-buildpacks in action.

```text
===> DETECTING
...
[detector] paketo-buildpacks/bellsoft-liberica 2.3.1
[detector] paketo-buildpacks/maven             1.2.0
[detector] paketo-buildpacks/executable-jar    1.2.1
[detector] paketo-buildpacks/apache-tomcat     1.1.1
[detector] paketo-buildpacks/dist-zip          1.2.1
[detector] paketo-buildpacks/spring-boot       2.2.4
===> BUILDING
[builder] Paketo Maven Buildpack 1.2.0
[builder] [INFO] Scanning for projects...
...
```

#### Push

Once the image is built, pushing it with Docker to Google Cloud Repository is exactly the same as pushing it to a custom container registry or the official and well know, DockerHub.

`docker push gcr.io/{GC_APP_ID}/app`

#### Deploy

Creating a Kubernetes cluster is pretty straightforward with `gcloud`.

`gcloud container clusters create app-cluster --num-nodes=2`

Using `kubectl` is possible to create a Kubernetes deployment. This will trigger the deployment of the image on the cluster previously created.

`kubectl create deployment app --image=gcr.io/{GC_APP_ID}/app`

#### Final steps

Containers on Google Kubernetes Engine are not accessible from the Internet because they do not have external IP addresses, unless the are explicitly exposed. Again with `kubectl`:

`kubectl expose deployment app --type=LoadBalancer --port 80 --target-port 8080`

Know it's time to get the external IP address to access your application.

`kubectl get service`

```text
NAME         CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
app          10.8.123.102    207.0.108.3     80:30877/TCP     15m
```

...and there you go! The application is up and running accepting connections on port 80 through the external IP provided.

### Conclusion

Every developer that had to build and maintain a Dockerfile, knows the struggle. The good news are that Buildpacks are a reality and this is adding big improvements in terms of consistency, performance and security to our builds. Another good point is that Spring Boot 2.3.0 will bring Buildpacks support out of the box - `pack` won't be necessary anymore and build process will be possible simply running `mvn spring-boot:build-image`.