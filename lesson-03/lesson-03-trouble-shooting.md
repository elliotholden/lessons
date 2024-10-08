# Lesson 3

## Trouble Shooting

## Permissions

You can use the __unshare__ sub command of podman to run a command in a *modified* user namespace. For example:

    podman unshare ls -dl /var/lib/mysql

<br>

### Useful trouble shooting commands

|Command| Description|Example|
|---------|-------------|--------|
|__podman top__|*Display percentage of CPU, memory, network I/O, block I/O and PIDs for one or more containers.*|__podman stats ctrID__|