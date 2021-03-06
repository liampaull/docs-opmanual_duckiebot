# Docker troubleshooting {#setup-troubleshooting-docker status=ready}

Assigned: OPEN

## I stopped all the containers and now Portainer or other basic containers are not available

You need to `ssh` in your Duckiebot and start the containers manually. 

Use `docker container list -a` to see its exact name and `docker start ![container_name]` to start it.

## I deleted all the containers 

You need to `ssh` in your Duckiebot and re-create the containers.

Note that the containers have some special options to be given.

The configuration is described in the YAML files in `/var/local', which currently are:

    /var/local/DT18_00_basic.yaml
    /var/local/DT18_01_health_stats.yaml
    /var/local/DT18_02_others.yaml
    /var/local/DT18_05_duckiebot_base.yaml

Each of this is in a [Docker compose][compose] format.

For example, `/var/local/DT18_00_basic.yaml` contains:

    version: '3'
    services:

      portainer:
        image: portainer/portainer:linux-arm
        command: ["--host=unix:///var/run/docker.sock", "--no-auth"]
        restart: always
        network_mode: "host"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock

      watchtower:
        image: v2tec/watchtower:armhf-latest
        command: ["--cleanup"]
        restart: always
        network_mode: "host"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock

You can now either run the container individually or use Docker compose.

Individually, you would copy the options:
  
    duckiebot $ docker run -d --restart always --network host -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer:linux-arm --host=unix:///var/run/docker.sock --no-auth
    
With Docker compose you would use:

    duckiebot $ docker-compose -f /var/local/DT18_00_basic.yaml up


[compose]: https://docs.docker.com/compose/



## Container does not start  {#setup-troubleshooting-docker-starting}

Symptom: `docker: Error response from daemon: Conflict. The container name "/![container_name]" is already in use by container "![container_hash]". You have to remove (or rename) that container to be able to reuse that name.`

Resolution: Stop (`docker stop ![container_name]`) if running and then remove (`docker rm ![container_name]`) the container with the  

## Docker exits with tls: oversized record received

If Docker exits with the above error when running remote comamnds, the most likely reason is different versions of Docker on your computer and Duckiebot. You can check that by running `docker version` on both devices. If that is indeed the case, you need to upgrade the Docker binaries on your computer. To do that, follow the official instructions [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/).
