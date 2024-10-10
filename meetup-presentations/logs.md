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

### httpd

### nginx
   * run a **nginx** container in detatched mode
        >podman run --name web -d -p 8080:8080 docker.io/library/nginx
   * In another terminal tail the logs with **podman logs -f**
        >podman logs -f web
   * Open a web brower and go to *http://localhost:8080*
        *notice that nothing is being output to the logs in the ssecond terminal window*
   * Go back to the first terminal window and use the **exec** command to restart the webserver
        >podman exec web nginx -s reload 

        *notice the log output in the second terminal window* 
