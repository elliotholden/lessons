# Logs

**Purpose:** The purpose of this lab is to demonstrate the difference between using
*podman logs* vs *tailing* application logs from within a container.

Whether or not the standard application logs are written to **stdout** and **stderr**
thus being accessible by __podman logs__ depends on how the log files are configured 
inside the container image. In standard __Apache__, non-containerized
applications, the application logs are typical writtend files located in __/var/log/httpd__.
But in containerized they are typically (but not always) symlinked to __/dev/stdout__
and __/dev/stderr__. So they don't actually end up in *files*, but rather they are sent
to the computer screen, where __stdout__ and __stderr__ are configure to display by
default on my Linux/Unix machines.

### Custom - local/httpd:latest
* Using a Containerfile create an image based on the __registry.access.redhat.com/ubi9__ image and install Apache (__httpd__). Make sure __httpd__ is started when the image is run. When building the image make sure to tag it as __localhost/httpd:latest__

          FROM registry.access.redhat.com/ubi9-minimal
          LABEL maintainer="Elliot Holden <elliot@ElliotMyWebGuy.com>" \
                    description="Container logs lab"

          RUN microdnf install --assumeyes --nodocs \
               --setopt="install_weak_deps=0" httpd && \
               microdnf clean all

          ENTRYPOINT ["httpd"]
          CMD ["-D","FOREGROUND"]

*  Build image:
   > podman build -t localhost/httpd:latest .

* Now run the newly created **httpd** container image in detatched mode
  >podman run --name custom-httpd -d -p 8888:80 localhost/httpd:latest

* In another terminal tail the logs with **podman logs -f**
  >podman logs -f custom-httpd

* Open a web brower and go to *http://localhost:8888*. Refresh the page a couple of times.

  *Notice that nothing is being output to the logs in the ssecond terminal window*

* Go back to the first terminal window and use the **exec** command to restart the webserver
  >podman exec web httpd -k restart 

  *Again, notice the log output in the second terminal window* 

### Official - docker.io/library/httpd:latest
* Now run an official Red Hat version of Apache (__httpd__) in detached mode.
  > podman run -d --name official-httpd -p 7777:80 registry.access.redhat.com/ubi9/httpd-24

* In another terminal tail the logs of this new container with **podman logs -f**
  >podman logs -f official-httpd

* Open a web brower and go to *http://localhost:7777*. Refresh the page a couple of times.

  *Notice there is activity in the logs*

* Go back to the first terminal window and use the **exec** command to restart the webserver

  *Again, notice there is activity in the logs*