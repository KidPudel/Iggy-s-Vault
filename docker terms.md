- `Dockerfile` – A text file containing a list of commands/set of instructions to call when creating a Docker Image. In our analogy this is the instructions to create the cookie cutter mould. So we have an application and we want to dockerize it. For that we create a Dockerfile, which will package up our application into an Image.

- `Docker Image` – A snapshot or a blueprint for creating containers. Images are _immutable_ and all containers created from the same image are exactly alike, if you need to change an image, you create a brand new image. In our analogy this is the cookie cutter mould.

- `Docker Container` – A single instance of the application with all of the dependencies it needs, that is live and running. In our analogy, this is a cookie.