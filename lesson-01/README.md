# Lesson 1
### What are conatiners?
Containers are a lightweight and portable way to deploy applications. They provide a consistent and reliable way to package an application and its dependencies into a single container that can be run on any system that supports containers, without requiring a specific environment or dependencies to be installed.

Unlike a __vm__, instead of virtualizing an entire machine, it virtualizes the application and its dependencies. This makes containers much more efficient and lightweight than traditional virtual machines.

__NOTE:__ Think of a container the same way as when you run 
>systemctl start httpd

You are running a *command* not a *virtual machine* 

When talking about containers there is the concept of a container *"image"* - the container __image__ is simply a tar file that holds the contents of a container. When you *run* a container you are actually running the container image. After you initially run a container from an image you can then *start* and *stop* the container.

### Registries
We also have the concept or __container registries__. A container *registry* is nothing more than a repository (or location) of where container images are stored. Some popular regisries are:

    - quay.io
    - docker.io
    - registry.redhat.io
    - regisry.access.redhat.com



#### Excersise 1.a
1. From your comman prompt type the following:
> podman images

If this is your first time using __podman__ or __docker__ you will notice there are no images.

2. Now type the following
> podman pull docker.io/library/httpd
