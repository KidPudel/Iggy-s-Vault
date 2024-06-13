Have been around since early days of Docker.
When you use bind mount, a file or directory on the host machine is mounted into a container
#### Characteristics:

- **Direct Access to Host Filesystem**: Bind mounts provide direct access to files and directories on the host, allowing for real-time updates.
- **No Docker Management**: Bind mounts are not managed by Docker, which means Docker doesn’t control their lifecycle.
- **Flexibility**: Useful for development environments where you need to test changes without rebuilding the Docker image.

#### Use Cases:

- **Development**: Mounting source code into a container to enable real-time development and testing.
- **Accessing Host Files**: When a container needs to interact with specific files on the host.