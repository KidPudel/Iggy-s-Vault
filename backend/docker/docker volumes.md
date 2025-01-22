Volumes are persistent data stores for containers, created and managed by Docker.
Created volumes stored within directory of a Docker host. When mounted this directory is what being mounted into. the container.
Similar to bind mounts, except volumes are managed by Docker and are isolated from the core functionality of the host machine.
Volumes are preferred mechanism for persisting data generated and used by Docker containers.
Unlike [[bind mounts]] that are dependent on the directory structure and OS of the host machine, volumes are completely managed by Docker.

- **Managed by Docker**: Volumes are created, managed, and stored by Docker, typically in `/var/lib/docker/volumes` on the host.
- **Independent of Host Directory Structure**: Since Docker manages them, they are not dependent on the host directory structure.
- **Backup and Restore**: Volumes are easier to back up and restore compared to bind mounts.
- **Access Control**: Volumes provide better isolation since they are managed by Docker, reducing the risk of unintentional modification from the host.

#### Use Cases:

- **Persistent Data Storage**: For databases, configurations, and other data that need to persist across container restarts.
- **Sharing Data Between Containers and/or host**: Volumes can be shared and reused among multiple containers.


So they allow for data persistence beyond lifecycle of a container.


can be created in docker or [[docker compose]]

```dockerfile
VOLUME /hostpipe
```

```yml
volumes:
	- ./hostpipe:/service/hostpipe
```