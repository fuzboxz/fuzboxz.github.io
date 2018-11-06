---
layout: post
title: Setting up Kali for Docker
categories: infosec
---

![Docker]({{"/content/posts/docker.png" | absolute_url }}){: .center}

Since I switched from Windows 10 to Linux, I stopped running virtual machines and merged most of my stuff to Docker. Compared to virtual machines, Docker containers are much faster, more convenient, take up less space and are easily disposable. Most of the products I used to run on VMs are available as Docker images, so instead of installing the OS AND the application, now I just run the container that has everything pre-configured and ready to go.

This post will be about installing and finetuning Kali on Docker and committing your changes. I won't go into too much detail about how Docker containers work and what's the difference between a VM and a Docker container, so I advise looking that up if you want more in-depth knowledge.

To get started you need to install [Docker CE](https://get.docker.com/) on your machine. Once you have docker installed you need to pull the official Kali container, start it and connect to the TTY.

    docker pull kalilinux/kali-linux-docker
    docker run -it kalilinux/kali-linux-docker /bin/bash

Congratulations, you are now running Kali in Docker. That was easy, wasn't it? Keep in mind, that this is not the same as the full Kali VM that you can download from the [Offensive Security](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/) website, just a minimal installation that still lacks the majority of the tools.

As with everything else, our first order of business is making sure that our container runs up-to-date packages.

    apt update && apt full-upgrade && apt auto-remove && apt auto-clean

As our container has very few tools installed by default, we need to figure out what tools to install next. As Kali has a lot of packages, installing everything is probably not the best way to go, unless you have plenty of time and space. Getting the *kali-linux-all* metapackage from the Kali repo can take up 15 gigabytes and even the Top10 package come at 3.5 gigs, so it's down to personal preference what to install. If you need everything, go with *kali-linux-all*, if you only want web stuff your pick should be *kali-linux-web*, but for general purposes I'm going to install *kali-linux-top10*. You can check out the available metapackages on the [Kali Linux website](https://www.kali.org/news/kali-linux-metapackages/).

    apt install kali-linux-top10

As the *kali-linux-top10* doesn't come with two of my favorite packages *nikto* and *searchsploit*, I will be installing these manually. You can pick and install your own packages.

    apt install exploitdb nikto

As *kali-linux-top10* comes with an unconfigured Metasploit, we also need to set that up before we have a full working environment. The first command will start the SQL server and the second command will initialize the Metasploit database(s).

    service postgresql start
    msfdb init

Once you have everything working, it's time to commit your changes to your own image, so you can pull up your preconfigured container whenever you need it. Run these command from your *host OS terminal* without closing the Kali container.

    docker ps -a
    docker commit <kali linux container id> kalipersonal
    docker images

If everything went right you should be able to see your customized Kali image among the Docker images. To start a new instance of the image, you just need to run the following command.

    docker run -it kalipersonal

Another way to do all of these steps automatically is to just create a *Dockerfile*, add the OS commands from above (convert to non-interactive when required) and build the image with *docker build*.

Stopped Docker containers can be restarted, but based on your needs, you might also want to configure *--mount* or *-v* for mounting folders. If you want to learn more about these options, please refer to the [Docker reference](https://docs.docker.com/engine/reference/commandline/run/).