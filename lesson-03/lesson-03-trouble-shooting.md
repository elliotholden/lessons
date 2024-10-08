# Lesson 3

## Trouble Shooting

## Permissions

You can use the __unshare__ sub command of podman to run a command in a *modified* user namespace. For example:

    podman unshare ls -dl /var/lib/mysql

This will run the **ls** command from the perspective of the container *namespace*

<br>

### Useful trouble shooting commands

|Command| Description|Example|
|---------|-------------|--------|
|__podman stats__|*Display percentage of CPU, memory, <br>network I/O, block I/O and PIDs for one<br> or more containers.*|__podman stats__ *ctrID*|
|__podman top__|Display the running processes of a container.|__podman top__ *ctrID pid user huser comm*|



The `USER` command in a Dockerfile sets the default user that will be used to run the container's main process. This user will be used to execute the `CMD` or `ENTRYPOINT` instruction in the Dockerfile.

When you use the `USER` command in a Dockerfile, you're setting the user ID (UID) and group ID (GID) that will be used to run the container's main process. This user will be used to own the process and its resources, such as files and network connections.

The `USER` command does not directly relate to the default user used when you "exec" into a container. When you use `docker exec` to run a command inside a container, you can specify a user using the `-u` or `--user` flag. If you don't specify a user, it will default to the __user__ set in the container's configuration, which is specified by the `USER` instruction in the Containerfile. If no `USER` instruction is set in the Containerfile, `podman exec` will default to using the root user (UID 0).

Regarding the `{{.User}}` field in the container image's JSON metadata, it refers to the user that was specified in the `USER` command in the Containerfile. This field is used to store the user ID and group ID that were set by the `USER` command.

Here's an example of how the `USER` command and the `{{.User}}` field relate:

Dockerfile:
```dockerfile
FROM ubuntu:latest
USER 1001:1001
CMD ["echo", "Hello World"]
```
In this example, the `USER` command sets the default user to `1001:1001` (UID:GID). When you build the image and inspect its metadata, you'll see the `{{.User}}` field set to `1001:1001`.

Container image metadata (JSON):
```json
{
  "User": "1001:1001",
  ...
}
```
When you run the container, the main process will run as the user `1001:1001`. If you use `docker exec` to run a command inside the container, you can specify a different user using the `-u` or `--user` flag.

### Exersise 3.a

1. Pull the Bitnami MariaDB image from docker and then run it.

        podman pull docker.io/bitnami/mariadb
    --

        podman run -d --name bitnami-mariadbc docker.io/bitnami/mariadb
        
2. Next type the command: **podman ps -a** and notice that the container is not running. Now, type the command: **podman logs bitnami-mariadb** and notice the error message telling you that the **MARIADBD_ROOT_PASSWORD** environment variable needs to be set. Remove the container and run it again with the evironment variable included in the command:

    **Remove:**

        podman rm bitnami-mariadb 

      --
      
      **Re-run:**


        podman run -d --name bitnami-mariadb -e MARIADB_ROOT_PASSWORD=password docker.io/bitnami/mariadb 

3. Now, **exec** into the container to run the ***whoami*** command to find out the default user.

        podman exec bitnami-mariadb whoami

    Notice it shows that are the user with the UID of **1001**

4. Next, again using **exec**, inspect the /etc/passwd file inside the container using **grep**

        podman exec bitnami-mariadb grep 1001 /etc/passwd

    **Output:**

    >1001:*:1001:0:container user:/:/bin/sh

    Notice this it does not show a username. This is because the username was never set when the user was created.

5. Now run the **podman top** command to show the the user inside the container that is running the main process (in this case **mariadbd**), and it also shows the UID that the container UID is mapped to on the host system. The **user** option is for the *container* user and the **huser** option is for the *host* user.

        podman top bitnami-mariadb user huser

    |USER|HUSER|
    |----|-----|
    |1001|101000|

<br>

__NOTE:__

* When you run `podman exec` without the `--user` option, it will use the default user set in the container's configuration, which is specified by the `USER` instruction in the Containerfile.
* If no `USER` instruction is set in the Dockerfile, `podman exec` will default to using the root user (UID 0).