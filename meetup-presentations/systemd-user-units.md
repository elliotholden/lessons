# {Systemd} user units
by Elliot Holden - elliot@ElliotMyWebGuy.com

__Purpose:__ This lab will demonstrate how __Podman__ can be used to generate a __Systemd user file__ which can be used to manage a container's runtime. You will also learn how to automatically update Systemd managed containers (*by using the __io.containers.autoupdate__ lable when initially running a container*). When using Systemd to manage a container's runtime the __enable__ subcommand of the __systemctl__ command can be employed to make sure that the container restarts on reboot of the host system.

* Before delving into Systemd we will first look at the *linger* feature of the **loginctl** command to understand how to make a command started by one user, *linger* around, even if the user is not logged in. This will come in handy when containers started by a particular user need to be persistant regardless of the user's login status.

* After that we will see how to restart the container automatically using the __--restart__ flag


## loginctl [enable-linger]
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

   >This is because __loginctl__ shows the users who are *logged in*. And since you became the user __jeff__ by opening a sub shell (*sudo su - jeff*), the user __jeff__ does not show in the output of the __loginctl__ command

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

   This time __jeff__ shows up in the output but his LINGER status is set to __no__. Now let us examine the effects of __jeff's__ LINGER status being set to __no__.

5. As __jeff__, run an nginx container while exposing port 7777 to your host system.

        podman run -d --name nginx -p 7777:80 docker.io/library/nginx

6. Open a web browser and view the page at: http://localhost:7777

7. After confirming the nginx default page, exit out of the SSH session for __jeff__ and try to vew the webpage again: http://localhost:7777 - notice it does not display. That is because exiting out of the __jeff__ users SSH session has also killed the container process that __jeff__ started. *Processes* that __jeff__ has started do not automatically *linger* around when he is not logged in.

8. Now SSH back into localhost as __jeff__, enable the LINGER feature of __loginctl__ and restart the __nginx__ container.

        ssh jeff@localhost

        loginctl enable-linger

        podman start nginx
        
   >__NOTE:__ A *username* can also be supplied as an argument to the __loginctl enable-linger__ command. In our example, it is ommitted because the *username* argument defaults to the *username* of the person who is running the command. The same goes with not having to be __root__ to run the command because we automatically have permission to run the command on ourself.

9. Now exit out the the user __jeff's__ SSH session. Attempt access the page: http://localhost:7777 - notice that it is succeful. Check the __loginctl list-users__ command and notice that __jeff's__ LINGER status is set to ***yes*** and his STATE says ***lingering***.

10. Lastly, restart the container engine's *host* server (or the guest virtual machine you are running podman inside). And try to access the web page http://localhost:7777 - notice that the web service if offline again. This is because we have not enable a restart policy for the nginx container. Let us see how to enable automatic restarts in the next section.


## podman run --restart always
Taking up where we left off in the last section lets enable the __podman-restart__ systemd service and update our container with the __--restart always__ option.

1. Make sure you are logged in as __jeff__ and enable __podman-restart.service__

        systemctl --user enable --now podman-restart.service

   >Alternatively you can login as root or a user with sudo privileges and run the following:

        sudo systemctl enable --now podman-restart.service

2. View the nginx container's current restart policy.

   __Method 1__ 

        podman inspect --format {{.HostConfig.RestartPolicy}} nginx

   __Method 2__

        podman inspect nginx | jq .[].HostConfig.RestartPolicy

3. Update nginx container with *--restart always*

        podman update --restart always nginx
        
   >__NOTE:__ The **--restart** flag is present in *podman version 5.2.2* but appears to be missing in previous versions. I have verified that it does not exist in *4.9.5* and *4.9.4*.

4. Make sure the nginx container is running and restart the host server

        podman start nginx

        sudo systemctl reboot now

   >__NOTE:__ *If you have not given sudo privileges to the user __jeff__, you need to exit out of the local SSH session for __jeff__ and make sure you are logged in with a user that has __sudo__ privileges in order to peform the reboot.*

5. Finally access the web service and notice that it should be running: http://localhost:7777


## Systemd
Now that we have seen how to use the Linger property of the __loginctl__ command and restart a container automatically with the __podman-restart__ service, let's look at managing our container's runtime with Systemd. We will use the __podman generate__ command to accomplish this task. For this task we can stop the __podman-retart__ service since managing our container's runtime as a Systemd service will allow it to restart automatically as long as we __enable_ it.

1. Login to your server as the user __jeff__ and stop the __podman-restart__ service. 

        systemctl --user disable --now podman-restart.service

2. Also remove the restart policy from the container since this will not be needed as well. 

        podman update --restart never nginx 

3. Mack sure the following directoy structure exists: __~/.confg/systemd/user__ 

        mkdir -p ~/.config/systemd/user

4. Change into the newly created directory and run the __podman generate systemd__ command to create the Systemd unit file. Then reload the Systemd daemon.

        podman generate systemd --name --new --files nginx

   >__NOTE__: View the __podman generate system --help__ to learn what the differenct options will do (--name, --new, --files)

        systemctl --user daemon-reload

5. Next, enable the newly created Systemd service.

        systemctl --user enable --now container-nginx.service

6. Verify the website service is back online by going to http://localhost:7777

7. Restart the host system and verify the service comes back online by going to http://localhost:7777


## Registry !!!!! This section is under contstruction !!!!!

6. Next, as __jeff__ run a registry container over port 5000.

        podman run -d -p 5000:5000 --name registry --restart always docker.io/library/registry

7. Updload the __nginx__ image to your newly created registry.

        podman push nginx localhost:5000/nginx
        
    __NOTE:__ You may get the following error

   >*Error: trying to reuse blob sha256:98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad at destination: pinging container registry localhost:5000: Get "https://localhost:5000/v2/": http: server gave HTTP response to HTTPS client*

   In which case you will need to add the __--tls-verify=false__ option to the __podman push__ command.

        podman push --tls-verify=false nginx localhost:5000/nginx

   Alternatively, you could create a file similar to **/etc/containers/registries.conf.d/my-registry.conf** with the contents:

        [[registry]] 
        location = localhost:5000
        insecure = true

8. Remove the localhost/nginx container

        podman rmi localhost/nginx

   Notice we are removing the container image located at ***localhost/nginx*** and not ***localhost:5000/nginx*** (our newly create registry *service*). The difference is that ***localhost/nginx*** image is "local" (on the host sytstem running our podman container engine). Whereas, in a real world application, ***localhost:5000/nginx*** would be a *remote* service (named something like __registry.my-company.io__)

8. As __jeff__, stop the currently running nginx container and run it again from the registry you just created. Expose port 8888 in the process. Make sure to include the label __io.containers.autoupdate=registry__ when running the container.
        
        podman stop nginx
-
        podman run -d -p 8888:8080 --name nginx --label "io.containers.autoupdate=registry" localhost:5000/nginx
