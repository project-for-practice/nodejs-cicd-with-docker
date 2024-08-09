# Running our docker image with docker Compose

Lets take a look at the docker compose file below.

```yaml
version: "3.8"
services:
  express_starter:
    container_name: express_starter
    image: user-name/express-starter:latest
    restart: always
    environment:
      - PORT=3030
    expose:
      - 3030
    ports:
      - 3030:3030
    networks:
      - express_nw
networks: # network for docker compose
  express_nw:
    driver: bridge
```

Here, we are running the image that we pushed earlier to docker hub. I am using the `repo name` and the tag from the above steps. You may change the image/container names and other fields like ports and other values accordingly.

Head over to your terminal and run the compose file with the following command:

> `docker compose up -d`

> <span style="colo:red;">Note: If you are using a private repository, you need to authenticate your docker hub account in the host/server where you are running the container. [Read more in this thread](https://stackoverflow.com/questions/31788256/how-to-pull-from-private-docker-repository-on-docker-hub)</span>.

This will run the container in the background. If check if the container is running or not you may run the following command:

### docker ps

To view the logs of the running container run:

> `docker logs <container id> or <container name>`

Example:

> docker logs express_starter

```bash
Output:

> express-starter@1.0.0 start
> node index.js

App listening on port 3030
> express-starter@1.0.0 start
> node index.js
App listening on port 3030

```

# Update the running image

We will be using a tool called `watchtower` to update our running containers. Add following line of code to our docker compose.

```yaml
watchtower:
  container_name: watchtower
  image: containrrr/watchtower
  environment:
    - WATCHTOWER_CLEANUP=true
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  command: --interval 30 <your-container-name>
```

Here, we are pulling the `watchtower image` and running it in our machine. You can specify as many container names you want separated by a space in the command key.

This watchtower container will scan the repository of specified container names every `30` seconds pull and replace the running container with a new image if it finds one.

The `WATCHTOWER_CLEANUP` environment variables ensures that the unused images will be automatically removed by `watchtower`, hence freeing up our storage space.

So our final docker compose file will be as follows:

```yaml
version: "3.8"
services:
  express_starter:
    container_name: express_starter
    image: user_name/image-name:tag
    restart: always
    environment:
      - PORT=3030
    expose:
      - 3030
    ports:
      - 3030:3030
    networks:
      - express_nw
watchtower:
  container_name: watchtower
  image: containrrr/watchtower
  environment:
    - WATCHTOWER_CLEANUP=true
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  command: --interval 30 express_starter
networks:
  express_nw:
    driver: bridge
```
