# Lesson 2
## Running Containers


### Entrypoint

The __entrypoint__ in a container is the default command that will execute when the container is run for the first time or (started after it has been stopped).

The __entrypoint__ can be set on the command line when initially *running* a container with the __--entrypoint__ option. Or in a Container file with the __ENTRYPOINT__ command.

**Command line example**

    podman run --entrypoint=/usr/local/bin/entrypoint.sh registry.redhat.io/ubi9

**Containerfile Example**:

    FROM registry.access.redhat.com/ubi9
    RUN yum install -y nmap
    ENTRYPOINT ["nmap 127.0.0.1"]


>__NOTE__:
It's not possible to change the entry point of a container image after it's been created and stopped.</p>
The entry point is specified in the Docker image's configuration, and once the image is created, it's immutable. When you run a container from an image, the entry point is set at that time, and it can't be changed later.</p>
Even if you stop the container, the entry point remains the same, and if you restart the container, it will use the same entry point.</p>
However, you can create a new container from the same image with a different entry point by using the --entrypoint flag when running the container again. For example:

    docker run -it --entrypoint=/new/entry/point myimage

*If you need to change the entry point of an existing container, you'll need to create a new image with the updated entry point and then create a new container from that image.*