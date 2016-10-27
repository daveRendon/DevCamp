# DevCamp
These guide will help instructors when delivering this course. There are some speaker notes in each deck, but this will provide a broader view of how the course was put together.

The Developer bootcamp is a collection of lectures, content and hands on labs that is intended to provide developers with a jumpstart to Azure development. There is an emphasis on the Platform as a Services (PAAS) offerings in Azure but there are some IaaS related topics as well.

The concept of "1-app, 3 ways" was born from the idea that developers want to see the whole picture, not just pieces. In previous training courses, they are focused on individual modules and learning objectives. For this bootcamp, we took the approach of a single application and functionality is added over the 2 days (Develop, deploy, monitor, scale). We also realized that developers may want to see the same functionality developed in different languages, so they can compare and contrast approaches.

The application is very simple; a CRUD based form with as few moving parts as possible, all while being industry/vertical agnostic. The application can be re-imagined in several ways with minimal impact to the session content and HOLs.
* Education - A science fair idea board
* Manufacturing - Process improvement
* E-Commerce - Item review/feedback


## Getting prepared to deliver ##
Working on a Windows machine is ok if you have Windows 10 Anniversary edition. This will allow you to show 

---
## Module 0 - Introduction, roadmap and course overview ###
In this session, we will provide a brief history of Azure, a quick overview of the capabilities available and introduction to the 2-day interactive workshop.

[View PowerPoint](Presentation/Module00-Overview.pptx?raw=true)

**Goal:** This deck is marketing focused and is designed to provide a high-level overview of Azure. It is geared towards newcomers to the Microsoft cloud offerings.

**Demos:** You can show the portal

**Session prep tips:** 
* Review the latest Azure features.
* Ensure that you are familiar with the roadmap sites on [Azure.com](https://azure.microsoft.com/en-us/updates/) and [Microsoft.com](https://www.microsoft.com/en-us/server-cloud/roadmap/) 

**Key takeaways:** 
* Ensure that the audience is prepared for 2 days of hands on development. If they are not developers, it may be difficult for them to understand the content 

**Common questions:**
* Will you be covering IaaS and/or Azure Stack? No.
* Will this cover competitive platforms? No.

**Watch out for:** Timing. This session is only 30 mins and is meant to level set the audience.

### HOL Proctoring ###
**Tasks to complete**
* Set up development Environment

**Exit criteria**
* It can take up to 20 minutes or more to deploy the resource group for development.

**Possible issues**
* No Credit card
* If they have an existing subscription
* If they cannot set up a trial


---

## Module 1 - Tools and Developer Environment Setup Overview ####
We will provide an overview of the developer tools available for developing on your platform.

[View PowerPoint](Presentation/Module01-DevTools.pptx?raw=true)

**Goal:** Provides an overview into the different development approaches, introduction to the SDKs, tools and frameworks for cross platform development.

**Possible demos:** 
* Azure CLI (on Windows 10 Bash, Docker, Linux/OSX, Windows)
* Azure PowerShell (On Windows)
* Visual Studio integration (On Windows)
* Java integration with Eclipse (On Linux/Windows)
* Node integration with VS Code (NPM on Windows/Linux/OSX) 

**Session prep tips:** 


**Key takeaways:**  

**Common questions:** 

**Watch out for:** 


### HOL Proctoring ###

**Tasks to complete**
* 

**Exit criteria**
* 

**Possible issues**
* 

----
##  Module 2 - Modern Cloud Apps Overview ####
We will provide an overview of some common cloud technologies, patterns and Azure features (Polyglot, scalability, app insights, Redis, patterns, traffic manager, global scale, blob, CDNs) and introduce you to the sample application. It is written 3-ways (.NET, Node.js and Java) so you can pick your platform of choice.

[View PowerPoint](Presentation/Module02-ModernCloudApps.pptx?raw=true)

**Goal:** 

**Demos:** 

**Session prep tips:**

**Key takeaways:**  

**Common questions:** 

**Watch out for:** 

### HOL Proctoring ###

**Tasks to complete**
* 

**Exit criteria**
* 

**Possible issues**
* 


---

##  Module 3 - Identity and Office365 APIs Overview ##
We will provide an overview of Azure AD, and discuss areas for integration with the Office 365 APIs.

[View PowerPoint](Presentation/Module03-Identity-0365Apis.pptxx?raw=true)

**Goal:** 

**Demos:** 

**Session prep tips:**

**Key takeaways:**  

**Common questions:** 

**Watch out for:** 

### HOL Proctoring ###

**Tasks to complete**
* 

**Exit criteria**
* 

**Possible issues**
* 

----

## Module 4 - DevOps Overview ####
We will provide an overview of Visual Studio Team Services (VSTS), DevOps concepts, build tasks, release environments, integration with Azure and Git/GitHub and Azure to create a cross-platform build, integration and release pipelines.

[View PowerPoint](Presentation/Module04-DevOps.pptx?raw=true)

**Goal:** 

**Demos:** 

**Session prep tips:**

**Key takeaways:**  

**Common questions:** 

**Watch out for:** 

### HOL Proctoring ###

**Tasks to complete**
* 

**Exit criteria**
* 

**Possible issues**
* 

---

## Module 5 - Infrastructure as code ####
Intro to Azure Resource manager and infrastructure as code.

[View PowerPoint](Presentation/Module05-ARM-IAC.pptx?raw=true)
**Goal:** 

**Demos:** 

**Session prep tips:**

**Key takeaways:**  

**Common questions:** 

**Watch out for:** 

### HOL Proctoring ###

**Tasks to complete**
* 

**Exit criteria**
* 

**Possible issues**
* 

---
## Module 6 - Monitoring ####
We will introduce you to the monitoring capabilities in Azure and show you how you can use them in your application.

[View PowerPoint](Presentation/Module06-Monitoring.pptxx?raw=true)

**Goal:** 

**Demos:** 

**Session prep tips:**

**Key takeaways:**  

**Common questions:** 

**Watch out for:** 

### HOL Proctoring ###

**Tasks to complete**
* 

**Exit criteria**
* 

**Possible issues**
* 


---
## Module 7 - Docker ####
In this module, we will provide an overview of Docker and introduce you to the Azure capabilities available for developers.

[View PowerPoint](Presentation/Module07-Containers.pptx?raw=true)

**Goal:** 
* Introduce audience to Docker
* Show Docker tools and integration with Azure and Windows

**Demos:** 
* Pulling an image, running a container
* Docker tools for Windows (Use the beta tools to switch from Windows and Linux on Windows 10)
* Docker tools for Visual Studio-preview
* App migration with Windows containers 
* Creating a Docker file

### Docker snippets reference ###
Search Docker for Microsoft Images
```
docker search microsoft
```

Pull a Docker image from Docker Hub (Linux Containers)
```
docker pull nginx
```

Pull a Docker image from Docker Hub (Windows Containers)
>**Warning, these take long to pull**
```
docker pull microsoft/aspnet
```

Docker run a container interactive
```
docker run -it -p 8000:8000 nginx bash

root@CONTAINERID:/# dir
root@CONTAINERID:/# exit

```
Docker view running containers
```
docker ps
```
Docker attach to running container
```
docker attach nginx
```
Docker stop a running container
```
docker stop nginx
```

Docker file

```
# The `FROM` instruction specifies the base image. You are
# extending the `microsoft/aspnet` image.
 
FROM microsoft/aspnet
 
# Next, this Dockerfile creates a directory for your application
RUN mkdir C:\app
 
# configure the new site in IIS.
RUN powershell -NoProfile -Command \
    Import-module IISAdministration; \
    New-IISSite -Name "ASPNET" -PhysicalPath C:\app -BindingInformation "*:8000:"
 
# This instruction tells the container to listen on port 8000. 
EXPOSE 8000
 
# The final instruction copies the site you published earlier into the container.
ADD bin/PublishOutput /app
```

> Create 25 NGINX containers (CMD/POSH)

```
for i in {1..25}; do docker run -p 80 -d nginx; done
```

> Delete all containers (BASH)

```
docker rm -f `docker ps --no-trunc -aq`
```

**Session prep tips:**
* https://github.com/harbur/docker-workshop
* https://github.com/RainBirdAi/docker-workshop
* On Windows
    * make sure you have the Anniversary edition
    * Activate the containers feature in Windows
    * Download the beta tools [Windows](https://docs.docker.com/docker-for-windows/)
* On MacOSX
    * Download the beta tools [MacOSX](https://docs.docker.com/docker-for-mac/)

**Key takeaways:**  
* Ensure the audience understands the difference between containers and Hyper-V virtualization
* Understand the difference between Linux and Windows containers
* Understand the container life-cycle process

**Common questions:** 
* How do I manage large container farms?
    * Azure container service, Docker Swarm, Mesosphere-DCOS, Kubernetes 

**Watch out for:** 
* Time to pull images. Do this ahead of time
* If you are demoing on a laptop, make sure you have enough space and resources. An 8 GB machine may not perform well if you have a lot of open applications running.

### HOL Proctoring ###
N/A No lab for this session

---
## Module 8 - Azure features and APIs ####
We will provide a quick lap around the various APIs, features and services available for developers.

[View PowerPoint](Presentation/Module08-Azure Features.pptx?raw=true)

**Goal:** 
To introduce the audience to a subset of the developer Azure platform features

**Demos:** 
* Cognitive Services
* Bot framework
*  

**Session prep tips:**

**Key takeaways:**  

**Common questions:** 

**Watch out for:**

### HOL Proctoring ###

N/A No lab for this session