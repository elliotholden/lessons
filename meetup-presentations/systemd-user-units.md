# {Systemd} user units
by Elliot Holden - elliot@ElliotMyWebGuy.com

__Purpose:__  Show how __Podman__ can be used to generate a __Systemd user file__ to start your containers. You will also learn how to automatically update Systemd managed containers.

1. Login to to any Linux system where __systemd__ is installed (Red Hat, AlmaLinux, Rocky Linux etc.) 

2.  Create a new linux user named __nina__ and give the user a password of *__password__*

        sudo useradd nina && sudo passwd nina
3. __ssh__ into localhost as __nina__
        
        ssh nina@localhost

    > __NOTE:__ *As root you may need to enable PasswordAuthentication in __/etc/ssh/sshd_config.d/40-disable-passwords.conf__* or the main __sshd_config__ file before you can ssh in with a password.

4. Create the directoy structure __.confg/systemd/user__ if it does not already exist.

        mkdir -p ~/.config/systemd/user

5. Create a Containerfile that uses docker.io/library/nginx as it's base image

        FROM docker.io/library/nginx
        LABEL maintainer="Your Name <your@emailaddress.com>"
              description="Systemd Lab"

    >podman build -t custom-nginx .

6. Next run a registry container over port 5000.

        podman run -d -p 5000:5000 --name registry docker.io/library/registry

5. As __nina__ run the container over port 8888. Make sure to included the __io.containers.autoupdate=registry__ when running the container.

        podman run -d -p 8888:8080 --name nginx docker.io/library/nginx
