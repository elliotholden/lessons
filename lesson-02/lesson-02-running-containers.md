# Lesson 2
## Running Containers


### Entrypoint

>__NOTE__:
It's not possible to change the entry point of a container image after it's been created and stopped.</p>
The entry point is specified in the Docker image's configuration, and once the image is created, it's immutable. When you run a container from an image, the entry point is set at that time, and it can't be changed later.</p>
Even if you stop the container, the entry point remains the same, and if you restart the container, it will use the same entry point.</p>
However, you can create a new container from the same image with a different entry point by using the --entrypoint flag when running the container again. For example: