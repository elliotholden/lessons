# {Kubernets} Ports EXPOSED
by Elliot Holden - elliot@ElliotMyWebGuy.com

__Purpose:__ This lab will demonstrate how Kubernetes ports are exposed and published. 

## ClusterIP Lab
To start with, we will first deploy an nginx application

1. Create a deployment called __web__ with *3 replicas* using the *nginx* docker image. Do not used the ___--port___ flag. Also use the declaritive form to create this deployment. Make sure to use the ***--save-config*** option

        kubectl create deploy web --image=docker.io/library/nginx --replicas=3 --save-config=true -o yaml --dry-run=client > web.yaml

        kubectl create -f web.yaml

2. Use __kubectl__ to check one of the pods and make sure **nginx** is actually running. *(HINT: You can also SSH into one of the cluster nodes and use the ***docker***, __crictl__, or __nsenter__ commands)*

        kubectl get pods

        kubectl exec web-564d578c4-9b4wt -- curl -s localhost:80

      >In the previous command, make sure to replace **web-564d578c4-9b4wt** with the actual name of one of the pods returned in the **kubectl get pods** command

3. Expose the __web__ depolyment as a ClusterIP service named __web-svc__. Do not use the __--port__ or __--target-pot__ options.

        kubectl expose deploy web --name web-svc 

   >Notice you have an **error: couldn't find port via --port flag or introspection**. The __expose__ command requires the  __--port__ option and if missing, will use the __port__ defined in the __deployment__. But you did not define a port with __--port__ when you initially created the the deployment. Thus you are getting the error. 

4. Used the __describe__ command to verify there is no __port__ defined in the deployment

        kubectl describe deploy web

5. Edit the __web.yaml__ declarative file you created in STEP #1, adding port __7480__ to the file. 

   >__HINT__: If you don't know the correct syntax to add to the file you can just re-run the __create__ command from STEP #1, but this time using the __--port__ option.

6. Apply the changes to the __web__ _deployment_ 

        kubectl apply -f web.yaml

    > It's very important that you initially used the __--save-config__ with the __create__ command in STEP #1 (I'ts not necessarily required in STEP #5). This is because the __apply__ command in STEP #6 requires that the initial deployment was created using the __--save-config__ option of the __create__ command or using the __apply__ command. Examine ___kubectl apply deploy -h___ for details.

7. Run STEP #3 again 

        kubectl expose deploy web --name web-svc 

      >Notice you get no errors... why? HINT: use *__kubectl expose -h__* to see why. Search for the string: ***--port=''*** inside the help and notice it says: *"Copied from the resource being exposed, if unspecified"*. The resource, which is the __web__ deployment now has a __port__, that you added in STEP #5. The __expose__ command uses this __port__ by default.

8.  Get the __node name__ of one of the cluster nodes. You can use the kubectl ***get nodes*** or ***get pods*** command.

        kubectl get nodes

        kubectl get pods -o wide

9. Get the clusterIP address of the __web-svc__ service by using __get service__

        kubectl get svc

10. SSH into one of the culster nodes. It doesn't matter *which* cluster node you SSH into. The cluster IP reachable from any node.

        minikube ssh -n devops-m03

    > Make sure to use one of the actual node names from *your* result of running ***get nodes*** and NOT my example node name: ***devops-m03***

11. Try to acces the nginx service using the __ClusterIP__ over port __7480__ (*use __telnet__ or __curl__ to test*). 

        curl 43.53.1.3:7480

      Notice you are getting an error

9. Describe the __web-svc__ service to see the issue. Use __kubectl edit__ to fix the issue

        kubectl describe web-svc

        kubectl edit web-svc

      >Understand that the issue was that the __TargetPort__ in the __web-svc__ service was not pointing to the _actual_ port the service is running on _inside_ the container. Because the __--target-port__ option was NOT used when creating the service, the __TargetPort__ of __7480__ was copied from the __--port__ option that was used when creating the service. And again, if there was no __--port__ used when creating the service, then it gets copied from the __--port__ that was used when creating the object being exposed, in this case the __deployment__. If there was no __--port__ used when creating the deployment, then you MUST specify the __--port__ when creating the service.

## NodeIP Lab