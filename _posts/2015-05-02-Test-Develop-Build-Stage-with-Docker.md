---
title: Test, Develop, Build, Stage with Docker
layout: post
category : posts
image: /assets/images/6204375064_795f1cd7d1_o_.jpg
tags : [docker, jenkins, continuous Integration]
---

_It doesn't matter whether you're a backend or a frontend ,when the sun comes up, you'd better be running_ `docker run`


Have you ever heard the [Joel Test](http://www.joelonsoftware.com/articles/fog0000000043.html)? It's a 12-questions test to check the quality of your software team.

>>
1. **Do you use source control?**
2. **Can you make a build in one step?**
3. **Do you make daily builds?**
4. Do you have a bug database?
5. Do you fix bugs before writing new code?
6. Do you have an up-to-date schedule?
7. Do you have a spec?
8. Do programmers have quiet working conditions?
9. Do you use the best tools money can buy?
10. Do you have testers?
11. Do new candidates write code during their interview?
12. Do you do hallway usability testing?

As you can see the first three questions are about the source control (aka CVS), the one step build process and the daily builds.
At Kataskopeo the joel score is 9/12, not so bad but we are working hard to grow up to the max level.

The impact of CVS, automatic builds and continuous delivery on our workflow is a critical factor, as a software house we need to check continuously the software quality, we develop new feature every day and we fix bugs across different versions of the same product.

During the last few years our products are grown very fast, today we use a lot of tool to develop client-server web applications. 
The frontend team needs nodejs and grunt to run tasks automatically, compass and sass to build the stylesheets and bower to install client side libraries, on the other hand the backend team needs composer, php (with some exensions) and an Oracle database instance to test and code the APIs.

We usually work in isolation so if I'm a frontend developer I don't need all the backend stuff, I develop unit tested components and when I want to see the result on the browser I use a stubbed API.<br>
This is **great** because during the development we do not need the whole product dependencies, but during the build process the full stack is required.
The build server needs to have installed Php, Composer, Node.js, NPM , Grunt, Bower, Compass, SASS and all the tools necessary to build the product; that means if you have a build server you need all that stuff installed and if you have _n_ projects with _n_ php versions you need _n_ build servers. 

**Every single change to our toolchain forces us to update the build server (ie: uninstall PHP55 and install PHP56) or maybe to set up a new one.**

To avoid that annoying problem we decided to rethink the internal CI infrastructure adopting [Docker](http://docker.com). Docker is a platform that enables a lightweight virtualization through [Linux Containers](https://linuxcontainers.org); it allows you to spin up a new virtualized invironment in few seconds.

As a backend dev I need to test an API against Oracle database; this is very simple, I just need to run:<br>
`docker run -d -p 49160:22 -p 49161:1521 wnameless/oracle-xe-11g` and I have a new fresh database instance.

## Current workflow
The current workflow adopted for all the team members is:

* Development
* Commit
* Push

After the `git push` a POST Hook is sent from [Gitlab](https://about.gitlab.com) to [Jenkins](https://jenkins-ci.org) and all the magic is done by the CI.


Jenkins is responsible to execute the following steps:

* Clone the project
* Install dependencies
* Run tests
* Compose the product package (**)
* Update the staging environment with the new code (**)

** if the Build was successful

As you can see the only think a developer should care about is to implement a new feature and push-it.<br>
The whole **Build** -> **Test** -> **Deploy-To-Stage** process is automated.

## Limitations
There are 2 main usage modes for jenkins; you can install a single instance and run all builds inside the same machine or you can connect the master jenkins node with a bounch of slave instances.


IMHO both the single instance and the master-slave solution have some limitations.
<br>
The first one forces you to install all you need inside the machine, so if you have projects with different requirements (for examample PHP54  and PHP55) you have to choose just one version.
The latter allows you to build and test packages with different environments but it may be hard to maintain.<br>
If you have a 10 slaves infrastructure every time a new PHP version is released you have to install the new package within a bounch of machines.

This is one of the main reason we were pushed to adopt the docker infrastructure.<br>
Docker is lightweight, you can handle software upgrade with a line of code, to upgrade php change the `FROM` directive within your Dockerfile

```docker
FROM php:5.6 # Previously php:5.5
```

## Jenkins + Docker
Our first target has been to keep the developer job unchanged then to maintain the current **Code** -> **Commit** -> **Push** workflow.<br>
Jenkins is a powerful Continuous Integration platform, we are using it from a very long time but the growing complexity of the projects forced us to manage an handful of VMs with a dozen of dependencies (such as ruby, nodejs, php, sass, etc..). It's very hard to maintain.

Anyway, we are confortable with that tool so we decided to redraw the game under the hood. We kept a master instance which should be responsible to monitor the build process running inside a throwaway containter.

#### How it works
- Jenkins receive the POST Hook from Gitlab;
- It clones the project;
- It runs and a tiny shell script (_build.sh_) which monitor the build process inside a container;

The **build.sh** does the magic with docker, let's see how...<br>


```bash
#!/bin/bash
docker-compose build
docker-compose run builder -u ant

## Copy the artifacts
docker cp `docker-compose ps -q builder`:/application/artifacts .

## Cleanup
docker-compose stop
docker-compose rm --force
```

We used the `docker-compose` orchestration tool to build e run containers.
Here is a configuration example.

**docker-compose.yml**

```yaml
builder:
  image: "k-team/builder:5.3"
# or "k-team/builder:5.4"
# or "k-team/builder:5.5" 
  user: jenkins
  volumes:
    - ".:/application"
    - "/cache:/cache"
  links:
    - db
db:
  image: wnameless/oracle-xe-11g
  ports:
    - "49160:22"
    - "49161:1521" 
```

We've created a bounch of base images `k-team\builder`, each one with a different php version, so it's easy to test product against PHP5.3 rather than PHP5.4 or PHP5.5.<br>
`docker-compose` will use that images to create the builder service.


We are so happy to play with docker because we are able to develop and test products without caring about the dependencies,<br> just run `git clone` and `docker-compose up` to spawn the necessary containers.
The same magic will run the Continous Integration and the Stage server.

![This is an happy dockerized developer](https://media.giphy.com/media/itDBteCsTFSVO/giphy.gif)