# Lesson 1
## What are conatiners?
Containers are a lightweight and portable way to deploy applications. They provide a consistent and reliable way to package an application and its dependencies into a single container that can be run on any system that supports containers, without requiring a specific environment or dependencies to be installed.

Unlike a __vm__, instead of virtualizing an entire machine, it virtualizes the application and its dependencies. This makes containers much more efficient and lightweight than traditional virtual machines.

To run a container you need a container runtime environment such as __containerd__ or __crio__. These container runtime enviroments are typically daemons that run in the background. Think of webserver software such as __Apache__ or database software such as __MariaDB__; they both run in the background. To manage containers you need a tool such as __Podman__ or __Docker__. These are sometimes refered to as container engines. For the purposes of this course we will be using Podman, however Podman is interchangeable with Docker. So any commands you learn in Podman should also work on Docker.

To install Podman do one of the following based on your operatring system

#### Excercise 1.a

Ubuntu / Debain

    sudo apt install podman

Red Hat / CentOS / Alma Linux

    sudo dnf install podman

__NOTE:__ Think of a container the same way as when you run any other service, like a webserver. 

Example:
>__systemctl start httpd__

When running a container you are running a *command* not a *virtual machine* 

When talking about containers there is the concept of a container *"image"* - the container __image__ is simply a tar file that holds the contents of a container. When you *run* a container you are actually running the container image. After you initially run a container from an image you can *stop* the container and then *start* the image again. Try those commands now.

#### Exercise 1.b

-- Run

  podman run --name web docker.io/library/httpd

-- Stop

    podman stop web

-- Start

    podman start web

-- List running containers

    podman ps 


### Registries
We also have the concept or __container registries__. A container *registry* is nothing more than a repository (or location) of where container images are stored. Some popular regisries are:

- quay.io
- docker.io
- registry.redhat.io
- regisry.access.redhat.com


#### Excersise 1.c

To search through your configured registries for any Apache (httpd) images type the following:

    podman search httpd

If you have some registries configured in /etc/containers/registries.conf you will see a plethora of results.

To limt and filter your results type the following: 

    podman search --limit 5 --format "table {{.Name}}" httpd


  >NAME  
   registry.access.redhat.com/rhscl/httpd-24-rhel7  
   registry.access.redhat.com/ubi9/httpd-24  
   registry.access.redhat.com/ubi8/httpd-24  
   registry.access.redhat.com/rhmap45/httpd  
   registry.access.redhat.com/rhmap44/httpd  
   registry.redhat.io/rhscl/httpd-24-rhel7  
   registry.redhat.io/ubi9/httpd-24  
   registry.redhat.io/rhel8/httpd-24  
   registry.redhat.io/ubi8/httpd-24  
   registry.redhat.io/rhel9/httpd-24  
   docker.io/library/httpd  
   docker.io/manageiq/httpd  
   docker.io/paketobuildpacks/httpd  
   docker.io/dockerpinata/httpd  
   docker.io/jitesoft/httpd  

*Notice that __15__ results are return and not __5__. That is because there are 3 configured registries in /etc/containers/registries.conf so __5__ results per registry are being returned*

From your comman prompt type the following:

    podman images


If this is your first time using __podman__ or __docker__ you will notice there are no images returned in the results.<br />


    [eholden@workstation-01 emwg]$ podman images\
    REPOSITORYTAG  IMAGE ID    CREATED     SIZE


Now type the following

    podman pull docker.io/library/httpd


### Inspecting Images
You can use the __skopeo__ command line tool inpsect an image on a remote registry before you pull it down. Simply typing: ```skopeo inspect docker://<FQCN>``` will pull down a JSON representation of the image meta-data. __FQCN__ stands for __Fully Qualified Container Name__ (*even though it's actually the "image" name*)


#### Excersise 1.d

To display the the FQCN __Name__ of the __AlmaLinux__ remote image, type the following:

    skopeo inspect -f 'Name: {{.Name}}' docker://almalinux/9-micro


To display the __Digest__ of the __Alpine Linux__ remote image, type the following:

    skopeo inspect -f 'Name: {{.Digest}}' docker://library/aplpine

To display the __Name__, __Size__ and __Digest__ of an image type the following at the command prompt:

    skopeo inspect -f 'Name: {{.Name}}\nSize: {{ range .LayersData }}{{ .Size }}{{ end }}\nDigest: {{.Digest}}' docker://almalinux/9-micro
