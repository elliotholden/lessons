# Lesson 3

## Trouble Shooting

## Permissions

You can use the __unshare__ sub command of podman to run a command in a *modified* user namespace. For example:

    podman unshare ls -dl /var/lib/mysql
