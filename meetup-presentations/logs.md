# {Container} Logs

**Purpose:** The purpose of this lab is to demonstrate how the **[podman|docker] logs** command is actually working.

Whether or not the standard application logs are written to **stdout** and **stderr**,
thus being accessible by __podman logs__, depends on how the log files are configured 
inside the container image. In standard __Apache__, non-containerized
applications, the application logs are typically written files located in __/var/log/httpd__.
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

  *Notice that nothing is being output to the logs in the second terminal window*

* Go back to the first terminal window and use the **exec** command to restart the webserver
  >podman exec custom-httpd httpd -k restart 

  *Again, notice that nothing is being output to the log in the second terminal window* 

### Official - registry.access.redhat.com/ubi9/httpd-24:latest 
* Now run an official Red Hat version of Apache (__httpd__) in detached mode.
  > podman run -d --name redhat-httpd -p 7777:80 registry.access.redhat.com/ubi9/httpd-24

* In another terminal tail the logs of this new container with **podman logs -f**
  >podman logs -f redhat-httpd

* Open a web brower and go to *http://localhost:7777*. Refresh the page a couple of times.

  *Notice there is an **ERROR** in the browser and no page is displayed. Also there is no activity in the logs.*

  *Troubleshoot the issue and create a new container if necesssary.*  **HINT**: use the **ss --pant** command to see which port the server is actually running on.

* After fixing the issue, again run the following a terminal:
  >pomdan logs redhat-httpd.

* Open a web brower and go back to *http://localhost:7777*. Refresh the page a couple of times.

  *You should now see activity in the __podman logs__*

* In another terminal (leave the terminal that is __podman logs__ open) use the **exec** command to restart the httpd webserver on the __redhat-httpd__ container.

  >podman exec redhat-httpd sh -c 'httpd -k restart'

  *Again, notice there is activity in the logs*

### Solution
* Shell into the __redhat-httpd__ container using the __exec__ command and inspect the contents of */etc/httpd/conf/httpd.conf*

  >podman exec -it redhat-httpd bash

  >vi /etc/httpd/conf/httpd.conf

     *Notice that both __CustomLog__ and __ErrorLog__ and being piped into /usr/bin/cat*


          ErrorLog |/usr/bin/cat

          #
          # LogLevel: Control the number of messages logged to the error_log.
          # Possible values include: debug, info, notice, warn, error, crit,
          # alert, emerg.
          #
          LogLevel warn

          <IfModule log_config_module>
          #
          # The following directives define some format nicknames for use with
          # a CustomLog directive (see below). 
          #
          LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
          LogFormat "%h %l %u %t \"%r\" %>s %b" common

          <IfModule logio_module>
               # You need to enable mod_logio.c to use %I and %O
               LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
          </IfModule>

          #
          # The location and format of the access logfile (Common Logfile Format).
          # If you do not define any access logfiles within a <VirtualHost>
          # container, they will be logged here.  Contrariwise, if you *do*
          # define per-<VirtualHost> access logfiles, transactions will be 
          # logged therein and *not* in this file.
          #
          #CustomLog "logs/access_log" common

          #
          # If you prefer a logfile with access, agent, and referer information
          # (Combined Logfile Format) you can use the following directive.
          #
          CustomLog |/usr/bin/cat combined
     </IfModule>

__TIP:__ You can also get the location of the logs by using the following command...
>httpd -S | grep -i log

### Extra

* For extra credit, run the Official Docker version of Apache and inspect the log files from within the container to see how logging has been implemented in this image.

  >podman run -d --name docker-httpd -p 9999:80 docker.io/library/httpd

  >podman exec docker-httpd __sh -c 'ls -l /var/log/httpd/*'__

  *Notice that ErrorLog and CustomLog are being sent to __/proc/self/fd/2__ and __/proc/self/fd/1__ respectively, which are both softlinks that utltimately point to __/dev/pts/0__* 

          root@f5fcd6f5488d:/usr/local/apache2/conf# pwd
          /usr/local/apache2/conf
          root@f5fcd6f5488d:/usr/local/apache2/conf# grep -iE 'CustomLog|ErrorLog' httpd.conf
          # ErrorLog: The location of the error log file.
          # If you do not specify an ErrorLog directive within a <VirtualHost>
          ErrorLog /proc/self/fd/2
               # a CustomLog directive (see below).
               CustomLog /proc/self/fd/1 common
               #CustomLog "logs/access_log" combined
          root@f5fcd6f5488d:/usr/local/apache2/conf# ls -l /proc/self/fd/1
          lrwx------. 1 root root 64 Oct 11 11:03 /proc/self/fd/1 -> /dev/pts/0
          root@f5fcd6f5488d:/usr/local/apache2/conf# ls -l /dev/stdout
          lrwxrwxrwx. 1 root root 15 Oct 11 10:14 /dev/stdout -> /proc/self/fd/1
          root@f5fcd6f5488d:/usr/local/apache2/conf# ls -l /dev/pts/0
          crw--w----. 1 root tty 136, 0 Oct 11 11:04 /dev/pts/0
          root@f5fcd6f5488d:/usr/local/apache2/conf# 

__TIP:__ From the command inside the container you can also use the following command to get the same results:
>httpd -S | grep -i log

__NOTE:__ In some configurations you may see that there are symlinks in /var/log/httpd pointing to /dev/stdout and /dev/stderr

     /var/log/httpd/access_log -> /dev/stdout  
     /var/log/httpd/error_log -> /dev/stderr

*In this scenario __/dev/stdout__ and __/dev/stderr__ may both point to __/proc/self/fd/1__ and __/proc/self/fd/2__ respectively wich, again, both point to __/dev/pts0__*