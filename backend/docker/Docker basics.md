Docker could be described, as an "_isolation tool_", that containerise an applications and their dependencies, for abstraction (standalone, independent, which makes it scalable), portability, scalability, versioning, etc.
# Virtual machine (VM) vs Docker
> We have a server. we can install on it virtual machines and run it. or..  
> We have a server. we install one operating system. and install a docker and create docker containers with different OS's.

### Virtual Machine
When we *create* a Virtual Machine configuring VM (choosing OS) and using provided installation media (like ISO) to install the OS, VM then emulates the whole hardware, like CPU, RAM, storage, network interfaces, and on top of that then the installed *whole* OS is running.
*Example:* If you create Windows 10 virtual machine, on your host Linux machine, VM would load an entire Windows 10 operating system, including the Windows kernel, device drivers, system files, etc.

## Docker
When running docker container ([[docker terms]]), docker creates an isolated environment "container", that *shares kernel, as well as system libraries and other core OS components* of the host OS they are running on.
Inside the container, Docker creates necessary user-space components like filesystems, process namespaces, network interfaces, etc., to provide isolation from the container and the host.
> NOTE: if the base image of the container, is OS or based on OS, that is different from host OS, then the process is slightly different.

Docker on host OS, will use a lightweight virtualization solution to run container with different OS.
For example if host is MacOS, and container is using Linux, then:
- Docker desktop provides lightweight Linux VM (e.g., Alpine Linux), and provides a Linux kernel environment for running Docker containers.
- Then Docker container communicates with Docker Engine that is running Linux VM, and this VM environment, where the actual Linux containers created and run.



![Docker](https://user-images.githubusercontent.com/63263301/227731841-7464eb9d-8de2-4159-8e98-0eebe5ba202c.png)


> Docker is a tool for creating and deploying isolated environments (read: virtual machines) for running applications with their dependencies.

# Docker components
A few terms you should be familiar with (including a baking analogy for ease of understanding):

[[docker terms]]




<img width="700" alt="image" src="https://github.com/KidPudel/backend-notes/assets/63263301/e9dafbd2-4b52-42b2-b53e-c8c57b5d53f8">  

<img width="700" alt="image" src="https://github.com/KidPudel/backend-notes/assets/63263301/8f392477-9fc1-4d4d-be47-76272dfe7b01">



# Why (as a data scientist) should I care?
Broadly, there are two use cases for Docker in ML:

`Run Only` (run): A run-only container means you edit your code on a local IDE and run it with the container so that your code runs inside the container. Here is one good example.  

`End-to-End Platform` (run and interact): An end-to-end platform container means you have an IDE or Jupyter Notebook / Lab, and your entire working environment, running in the container, and also run the code inside it (with the exception of the working file system which can be mounted).
We will focus on the second use case.

Reasons to use Docker in data science projects
Using docker containers means you don't have to deal with "works on my machine" problems.
Generally, the main advantage Docker provides is standardization. This means you can define the parameters of your container once, and run it wherever Docker is installed. This in turn provides two major advantages:

Reproducibility: Everyone has the same OS, the same versions of tools etc. This means you don't need to deal with "works on my machine" problems. If it works on your machine, it works on everyone's machine.
Portability: This means that moving from local development to a super-computing cluster is easy. Also, if you're working on open source data science projects, like we do at DAGsHub, you can provide collaborators with an easy way to bypass setup hassle.
Another huge advantage – learning to use Docker will make you a better engineer, or turn you into a data scientist with super powers. Many systems rely on Docker, and it will help you turn your ML projects into applications and deploy models into production.


docker is reproducing envioronments

by crrating a dockerfile tgat defines a blueprint for images
images are a tamplets for running a container
container is a running process like some application with defined environment 


# When to use it?
Use docker when you need to host something, isolate (to be indpendent, in self-containing environment), easily share and maintain.

Imagine you have a software, that has dependencies or versions, so you want to ensure that software is running consistently in any enviornment, therefore using Docker will isolate application and its dependencies in containers, making it not rely on it's dependencies.
- Isolation (not rely on dependencies)
- Portability (because it encapsulates)
- Microservices: Encapsulating power of docker, lets you isolate each part of the software, making it not depend on each other, making it easier to manage, scale and update individually
- Versioning: Docker images can be versioning, making it easy to roll back
- CI/CD: Docker provides consistent and reliable abstracted environment, that makes it easy to work with
- Efficient: Containers shares a host OS kernels, unlike traditional virtualization
- Simplification: Since we can encapsulate an app in it's own environment with it's dependencies already set there, it is easy to setup for devs
- Easy to replace containers and adding new ones (for horizontal scaling)

# Analogy
Imagine, you are living in a big grey city, and your memory of what the nature looks like, what was the color of the leaves starting to fade.
Then, I pay you a visit, and i show you my magical camera  with the brand name **_Docker_** that can take a pictures... or **_images_**.
You become very curious about it.  
Then i tell you that recently i've visited Switzerland and took some **_images_** for you.  
You take it and stick it to the fridge... and stand in front of it. Then magic happens, you go through the image, and now you suddenly IN the Switzerland... all those beautiful mountains, crystal clear lakes, greenlandscapes, almost like a candy.  
So instead of acquiring a Visa, buying a flight tickets, getting a car to get to that view, instead with a help of docker's image, you use everything that is already configured/ready for you to use, and you are good to go.
