

It seems that I was incorrect in my previous response.

When you run `podman exec CtID whoami`, the `whoami` command is executed inside the container `CtID`, and the output shows the username that is currently set as the default user for the container.

In your case, it seems that `podman exec` is indeed using the `USER` instruction set in the Dockerfile (or Containerfile) as the default user for the command, even if you don't specify the `--user` option explicitly.

This behavior is actually consistent with the documentation for `podman exec`, which states that if no user is specified, `podman exec` will use the user specified in the container's configuration, which is set by the `USER` instruction in the Dockerfile.

So, to summarize:

* When you run `podman exec` without the `--user` option, it will use the default user set in the container's configuration, which is specified by the `USER` instruction in the Dockerfile.
* If no `USER` instruction is set in the Dockerfile, `podman exec` will default to using the root user (UID 0).

Thank you for pointing out my mistake earlier!