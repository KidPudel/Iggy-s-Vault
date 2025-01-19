
We can create a user-defined bridge between 2 and more containers to run in a shared network or docker network.  
Create a docker network:

```bash
 docker network create -d bridge my-net
 docker run --network=my-net -itd --name=container3 busybox
```

If those containers share a docker network, then when specifying hostname / address in database, we can simply put name of a docker network


# [Drivers](https://docs.docker.com/engine/network/#drivers)

The following network drivers are available by default, and provide core networking functionality:

| Driver    | Description                                                              |
| --------- | ------------------------------------------------------------------------ |
| `bridge`  | The default network driver.                                              |
| `host`    | Remove network isolation between the container and the Docker host.      |
| `none`    | Completely isolate a container from the host and other containers.       |
| `overlay` | Overlay networks connect multiple Docker daemons together.               |
| `ipvlan`  | IPvlan networks provide full control over both IPv4 and IPv6 addressing. |
| `macvlan` | Assign a MAC address to a container.                                     |



