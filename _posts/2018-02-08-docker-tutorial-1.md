---
title: 'Conducting reproducible research with Docker (Part 1 of 3)'
date: 2018-02-09
permalink: /posts/2018/02/docker-tutorial-1/
tags:
  - Research
  - R
  - Reproducibility
---

In scientific research, reproducibility is a necessary (though not sufficient) condition for validity. But conducting reproducible research is hard! Sadly, many psychological studies [fail](As  a psychologist, when we conduct a study, we believe that researchers ought to be able to re-run your experiments or studies and obtain similar results (empirical reproducibility)) tests of empirical reproducibility.  Unfortunately, there's no software package that can solve the set of structural and statistical issues likely at the root of those non-replications. 

Still, there are some tools that can help us achieve statistical or computational reproducibility. This kind of reproducibility means that another researcher can take _our data_ and reproduce the analyses we conducted in a published paper. Sadly again, many studies in psychology [fail here too](https://www.nature.com/news/stat-checking-software-stirs-up-psychology-1.21049). However, here the problem really might be solved with better tools--tools like [R Markdown](https://rmarkdown.rstudio.com/) that can help ensure that our results sections are reflective of our actual analyses.

Here I'm going to describe another tool for producing statistically and computationally reproducible research, Docker. Reproducibility demands we make available the data and analyses scripts used in our research projects, but sometimes the line between our personal computer systems and our projects can start to blur. Our projects have "dependencies" that are required for them to run properly. So, to ensure other researchers can reproduce our projects, we need to clue them in to these dependencies in some way. The simplest way would be to dump our `sessionInfo()` at the bottom of the page. That's easy in the moment, but not easy down the road for those who want to reproduce our work. The easier we can make reproducible research the whole way through, the better. Keep in mind, the researcher most likely to attempt to reproduce your work is _future you_. 

Here, I'll show you how to use Docker to create reproducible workflows for scientific research.

# What is Docker?

![docker logo](https://www.docker.com/sites/default/files/horizontal.png)

Docker is a tool for making containerized applications. The docker engine is like a very lightweight virtual machine engine. A virtual machine is (to oversimplify) a computer program that simulates another computer system, typically another operating system. This allows you to run a windows app on your mac, or a linux progam on windows, and so forth.

Docker creates a "containerized" version of an application that includes everything needed to run the app: OS, headers, libraries, packages, etc. This container is saved as an "image", that can shared with others. This allows people working on different computers, with different OS versions, package versions, etc to share and execute code or apps. So long as you have Docker installed on your computer and the right Docker image, you can spin up a container that will exactly reproduce the environment needed for the app, no matter what your own personal computing environment looks like.

Maybe you're seeing how this can help us do reproducible research: if we create a containerized version of R, we can ensure we have R, R packages, system libraries, etc all in the right versions to reproduce the analyses. And because everything is held together in the container, if we share the image with another researcher, or with our future selves, it won't matter that they might have a different computer with different OS, packages, etc.

## The Rocker Project

Carl Boettiger and Dirk Eddelbuettel at [The Rocker Project](https://www.rocker-project.org/) have done the hard work of properly organizing R and RStudio server applications into well-maintained and versioned Docker images. These Docker containers run R and RStudio server with sensible security options and some helpful base packages. Their website is also a good resource for help using these images. In this tutorial, we'll use their image to run RStudio server in a Docker container. In Part 2, we'll build off their work to make our own image with whatever packages we like.

## Docker vs. Packrat

There's another solution to the problem of statistical and computational reproducibility in R, called [packrat](https://rstudio.github.io/packrat/). I will admit I don't have a great deal of familiarity with packrat, but I can discuss some differences. First, packrat is focused on R and R alone. This means if you incorporate other languages (e.g., python) in your projects, you will need multiple reproducibility solutions. In contrast, Docker handles everything. Second, Docker gives us other nice and powerful features, such as an easy way to run code remotely on cloud servers. Finally, there's nothing stopping you from using both approaches (even using packrat inside your Docker container).

# Tutorial

In this tutorial we'll ...

1. Install docker
2. Make a docker cloud account
3. Run a docker image from docker cloud

_Along the way, I'll assume you are comfortable using the terminal. I'll also assume you're on a mac, though things shouldn't be that different on Linux. I can't really speak to Windows, but the overall process should be similar._

## 1. Installing Docker (on Mac)

I'd advocate for installing Docker on Mac using [homebrew](https://brew.sh/). If you don't have homebrew, Docker has an [installation guide for mac](https://docs.docker.com/docker-for-mac/install/) that covers all the steps to install the traditional way. 

To install using homebrew, open up terminal and run:

```bash
brew update && brew cask install docker
```

Then launch docker from your applications (or with spotlight, cmd-space and type "docker"). You'll need to enter your administrator password.

_Optional_: set up bash completion for docker by running the below commands in terminal:

```
brew install bash-completion
brew install docker-completion
brew install docker-compose-completion
brew install docker-machine-completion
```

## 2. Make your Docker cloud account

When you first launch Docker it should prompt you to sign in or create a Docker cloud account. Alternately, you can go to [hub.docker.com](https://hub.docker.com/) and create an account there. Dockerhub is a centralized store for docker images (saved containers). In the next step, we'll grab an image from dockerhub to run a container our machine. Eventually, this will host your own personalized docker images (part 2 of this series). 

In the next step, we'll load the [tidyverse container](https://hub.docker.com/r/rocker/tidyverse/) from the rocker project's page on dockerhub. 

## 3. Run a docker image from docker hub

Ok, now let's actually get a docker container image running on our machine. First, make sure Docker is running on your machine (check the menubar for the icon). Then, head back over to terminal and enter the following command:

```bash
docker run -d -p 8787:8787 -v "`pwd`":/home/rstudio/working -e PASSWORD=rstudio -e ROOT=TRUE rocker/tidyverse:3.4.3
```

There's a lot going on in here so let's break down this command. 

1. The first part, `docker run` says we want to start running a docker container. 
2. The `-d` flag tells the container to run in the background (detached)
3. The `-p 8787:8787` flag maps port from inside the docker container to the main computer. This container will end up running an instance of RStudio server, which will be available at `localhost:8787`. Port 8787 happens to be the default, but it can be nice to be explicit. 
4. The ```-v `pwd`:/home/rstudio/working``` flag uses the --volume tag to connect the filesystem on our machine to our docker container. It maps our present working directory to a folder in the docker container called "working" that's in a location we can access through the RStudio interface. This lets you access whatever data or project files you need from your computer in the docker container.
5. The `-e PASSWORD=rstudio` flag sets an environment variable "PASSWORD" to "rstudio". This sets the password to access the rstudio server instance. Here we're just explicitly setting the password to the default, "rstudio". If you run this remotely (part 3 teaser!), this should obviously be changed.
6. The `-e ROOT=TRUE` flag gives us root access from inside RStudio. This can be helpful for installing linux dependencies when installing R packages.
7. Finally `rocker/tidyverse:3.4.3` specifies the docker image to run. That is, version 3.4.3 of the rocker/tidyverse image. If we didn't specify a tag, docker would default to the "latest" tag.

When you run a container without its image present locally, Docker will automatically download it.

## Using RStudio

Now open up your browser and navigate to `localhost:8787`. Enter "rstudio" as your username and whatever password you set as the password (defaults to "rstudio").

![sign in](/images/rstudio-sign-in.png) 

You will then be met with a fully-functioning RStudio interface. In the lower right you should see the file browser with the "working" directory we mapped when we ran the container. 

![file pane](/images/rstudio_interface.png) 

If you make changes to files in "working" inside this Docker container, they will also be reflected on your computer's file system.

Feel free to play around with this, you can see the already-installed R packages by typing `sessionInfo()`. 

## Configuring your container

Finally, you may need to adjust how much of your machine you allow Docker to use. On mac, Docker is very "polite" so it doesn't give itself very much of your machine's resources. But, because you plan to be working in this container, you will probably want to give it some more juice.

![docker preferences](/images/docker_prefs.png)

To fix this, access the docker preferences via the menu button and select the "advanced" tab. Then, adjust to your liking. There doesn't seem to be any harm to letting docker have full access to your system resources, at least not when used in this fashion.

## Coming up next ...

That's it for Part 1 of this series. Next, in Part 2 we'll discuss customizing a docker image with your own personal R environment. Till then you might want to poke around a bit and see what's available on dockerhub. I won't cover it's use in this series, but if you do any work in python, the [jupyter notebook datascience container](https://hub.docker.com/r/jupyter/datascience-notebook/) is worth checking out.