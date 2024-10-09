# Logs

1. Demonstrate the difference between using **podman logs** vs *tailling* application logs from within a container. Use the **nginx** container to show this.

    * run a **nginx** container in detatched mode
        >podman run --name web -d -p 8080:80 docker.io/library/nginx
    * In another terminal view the logs with **podman logs**
        >podman logs web

    * Open a web brower and go to *http://localhost:8080*
        *notice that nothing is being output to the logs in the ssecond terminal window*

    * Go back to the first terminal window and use the **exec** command to restart the webserver
        >podman exec web httpd -k restart

        *notice the log output in the second terminal window* 