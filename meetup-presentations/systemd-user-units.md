# {Systemd} user units
by Elliot Holden - elliot@ElliotMyWebGuy.com

__Purpose:__ This lab will demonstrate how __Podman__ can be used to generate a __Systemd user file__ which can be used to manage a container's runtime. You will also learn how to automatically update Systemd managed containers (*by using the __io.containers.autoupdate__ lable when initially running a container*). When using Systemd to manage a container's runtime the __enable__ subcommand of the __systemctl__ command can be employed to make sure that the container restarts on reboot of the host system.

Before delving into Systemd we will first look at the *linger* feature of the **loginctl** command to understand how to make a command started by one user, *linger* around, even if the user is not logged in. This will come in handy when containers started by a particular user need to be persistant regardless of the user's login status.

### loginctl [enable-linger]
To start with, we will first examine the workings of __loginctl__ and it's subcommand __enable-linger__

1. Login to your Linux system as yourself and create a new user named __jeff__.

        sudo useradd jeff

   Switch to this new user

        sudo su - jeff

2. As __jeff__ run the __loginctl list-users__ command.

        loginctl list-users

   >At this point the output only shows *your* username and not __jeff__.

    |UID|USER|LINGER|STATE|
    |---|---|---|---|
    |502|elliot|yes|active|

   >This is because __loginctl__ shows the users whow are *logged in*. And since you became the user __jeff__ by opening a sub shell (*sudo su - jeff*), the user __jeff__ does not show in the output of the __loginctl__ command

3. Exit out of the sub shell, create a password for __jeff__, and this time *login* as __jeff__ by SSH'ing to localhost.

        sudo passwd jeff

        ssh jeff@localhost

    > __NOTE:__ *As root, you may need to enable PasswordAuthentication in __/etc/ssh/sshd_config.d/40-disable-passwords.conf__* or the main __sshd_config__ file before you can ssh in with a password.

4. After logging in as __jeff__ run the __loginctl__ command again.

        loginctl list-users

    |UID|USER|LINGER|STATE|
    |---|---|---|---|
    |502|elliot|yes|active|
    |1001|jeff|no|active

   This time __jeff__ shows up in the output the his LINGER status is set to __no__. Now let us examine the effects of __jeff's__ LINGER status being set to __no__.

5. As __jeff__, run an nginx container while exposing port 7777 to your host system.

        podman run -d --name nginx -p 7777:80 docker.io/library/nginx

7. Open a web browser and view the page at: http://localhost:7777

8. After confirming the nginx default page, exit out of the SSH session for __jeff__ and try to vew the webpage again: http://localhost:7777 - notice it does not display. That is because exiting out of the __jeff__ users SSH has also killed the container process that __jeff__ started. *Processes* that __jeff__ has started do not automatically *linger* around when he is not logged in.

9. Now SSH back into localhost as __jeff__, enable the LINGER feature of __loginctl__ and restart the __nginx__ container.

        ssh jeff@localhost

        loginctl enable-linger

        podman start nginx
        
   >__NOTE:__ A *username* can also be supplied as an argument to the __loginctl enable-linger__ command. In our example, it is ommitted because the *username* argument defaults to the *username* of the person who is running the command. The same goes with not having to be __root__ to run the command because we automatically have permission to run the command on ourself.

10. Lastly, exit out the the user __jeff's__ SSH session. Attempt access the page: http://localhost:7777 - notice that it is succeful. Check the __loginctl list-users__ command and notice that __jeff's__ LINGER status is set to ***yes*** and his STATE says ***lingering***.

### Systemd
1. Login to to any Linux system where __systemd__ is installed (Red Hat, AlmaLinux, Rocky Linux etc.) 

2.  Create a new linux user named __nina__ and give the user a password of *__password__*

        sudo useradd nina && sudo passwd nina
3. __ssh__ into localhost as __nina__
        
        ssh nina@localhost

    > __NOTE:__ *As root you may need to enable PasswordAuthentication in __/etc/ssh/sshd_config.d/40-disable-passwords.conf__* or the main __sshd_config__ file before you can ssh in with a password.

4. Create the directoy structure __.confg/systemd/user__ if it does not already exist.

        mkdir -p ~/.config/systemd/user

5. Create a Containerfile that uses docker.io/library/nginx as it's base image

        FROM docker.io/library/nginx \
        LABEL maintainer="Your Name <your@emailaddress.com>"
              description="Systemd Lab"

    >podman build -t custom-nginx .

    * At this point, running __podman images__ should show the container: __localhost/custom-nginx latest__ in the results.

6. Next run a registry container over port 5000.

        podman run -d -p 5000:5000 --name registry docker.io/library/registry

7. Updload the __custom-nginx__ image to your registry

        podman push custom-nginx localhost:5000/custom-nginx
        
    __NOTE:__ You may get the following error

   >*Error: trying to reuse blob sha256:98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad at destination: pinging container registry localhost:5000: Get "https://localhost:5000/v2/": http: server gave HTTP response to HTTPS client*

   In which case you will need to add the __--tls-verify=false__ option to the __podman push__ command.

        podman push --tls-verify=false custom-nginx localhost:5000/custom-nginx

   Alternatively, you could create a file similar to **/etc/containers/registries.conf.d/my-registry.conf** with the contents:

        [[registry]] 
        location = localhost:5000
        insecure = true

8. Remove the localhost/custom-nginx container

        podman rmi localhost/custom-nginx

8. As __nina__ run the custom-nginx container from the registry you created over port 8888. Make sure to included the __io.containers.autoupdate=registry__ when running the container.

        podman run -d -p 8888:8080 --name nginx localhost:5000/nginx
