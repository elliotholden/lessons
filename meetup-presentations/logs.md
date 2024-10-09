# Logs

Demonstrate the difference between using **podman logs** vs *tailling* application logs from within a container. Whether or not the standard application logs are written standard out and standard eorr all depends on how the log files are configured. In __Apache__ containers the __/var/log/httpd__ logs are not written to standard out and standard error. But in __nginx_ they are written to *standard out* and *standard error* because /var/log/nginx/access.log is symlinked to /dev/stdout and /var/log/error.log is symlinked to /etc/dev/stderr.



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
