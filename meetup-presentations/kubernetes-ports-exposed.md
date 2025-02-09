# {Kubernets} Ports EXPOSED
by Elliot Holden - elliot@ElliotMyWebGuy.com

__Purpose:__ This lab will demonstrate how Kubernetes ports are exposed and published. 

## kubectl deploy 
To start with, we will first deploy an nginx application

1. Create a deployment called __web__ with *3 replicas* using the *nginx* docker image. Do not used the ___--port___ flag. Also used the declaritive form to create this deployment. Make sure to use the ***--save-config*** option

        kubectl create deploy web --image=docker.io/library/nginx --replicas=3 --save-config=true -o yaml --dry-run=client > web.yaml

        kubectl create -f web.yaml

2. Use __kubectl__ to check one of the pods and make sure **nginx** is actually running.

        kubectl get pods

        kubectl exec web-564d578c4-9b4wt -- curl -s localhost:80

      >In the previous command, make sure to replace **web-564d578c4-9b4wt** with the actual name of one of the pods returned in the **kubectl get pods** command

3. Expose the __web__ depolyment as a ClusterIP service named __web-svc__. Do not use the __--port__ or __--target-pot__ options.

        kubectl expose deploy web --name web-svc 

   >Notice you have an **error: couldn't find port via --port flag or introspection**. The __expose__ command requires the  __--port__ option and if missing, will use the __port__ defined in the __deployment__. But you did not define a port with __--port__ when you initially created the the deployment. Thus you are gettign the error. 

4. Used the __describe__ command to verify there is no __port__ defined in the deployment

        kubectl describe deploy web

5. Edit the declarative file you created in STEP #1, adding port __7480__ to the file. 

   >__HINT__: If you don't know the correct syntax to add to the file you can just re-run the __create__ command from STEP #1, but this time using the __--port__ option.

6. Apply the changes to the __web__ _deployment_ 

        kubectl apply -f web.yaml

    > It's very important that you initially used the __--save-config__ with the __create__ command in STEP #1 (I'ts not necessarily required in STEP #5). This is because the __apply__ command requires that the initial deployment was created using the __--save-config__ option of the __create__ command or using the __apply__ command. Examine ___kubectl apply deploy -h___ for details.

7. Run STEP #3 again 

        kubectl expose deploy web --name web-svc 

      >Notice you get no errors

8. As __jeff__, run an nginx container while exposing port 7777 to your host system.

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
