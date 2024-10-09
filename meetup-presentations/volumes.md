# Volumes

1. Demonstrate the difference between a **named volume** and a **bind mount**

    __named volume__ using *-v* option
    >podman run -v data-volume:/data docker.io/library/nginx

    __bind mount__ using the *--mount* option
    >podman run --mount type=bind,src=/home/elliot/db_data:/var/lib/mysql docker.io/library/mariadb:**z**
