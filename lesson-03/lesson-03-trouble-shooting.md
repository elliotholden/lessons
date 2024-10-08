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

### Exersise 3.a

1. Pull the Bitnami MariaDB image from docker and then run it.

        podman pull docker.io/bitnami/mariadb
    --

        podman run -d --name bitnami-mariadbc docker.io/bitnami/mariadb
        
2. Next run **podman ps -a** and notice that the container is not running. Run **podman logs bitnami-mariadb** and notice the error message telling you that the **MARIADBD_ROOT_PASSWORD** environment variable needs to be set. Remove the container and run it again with the evironment variable included in the command:

        podman rm bitnami-mariadb 

      --

        podman run -d --name bitnami-mariadb -e MARIADB_ROOT_PASSWORD=password docker.io/bitnami/mariadb 



