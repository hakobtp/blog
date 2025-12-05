# Managing Containers

As you start using Docker, you will create many containers over time. It is important to know 
which containers are running, which are stopped, and how to manage them.

## Listing Running Containers

To see the containers currently running on your system, use this command:

```bash
docker container ls
```

The output will look something like this:

```text
CONTAINER ID   IMAGE           COMMAND              CREATED        STATUS          PORTS         NAMES
b8f1e0b80096   pgadmin4        "/entrypoint.sh"     3 months ago   Up 15 mins     80/tcp        pgadmin4
a456acd23d64   postgres        "docker-entrypoint"  3 months ago   Up 15 mins     5432/tcp      postgres
f19bec238414   mongo           "docker-entrypoint"  8 months ago   Up 15 mins     27017/tcp     mongodb
```

Here is what each column means:

| Column           | Description                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------ |
| **Container ID** | A unique ID for the container.                                                             |
| **Image**        | The image the container was created from.                                                  |
| **Command**      | The command that runs inside the container.                                                |
| **Created**      | When the container was created.                                                            |
| **Status**       | Current state: created, running, exited, or paused.                                        |
| **Ports**        | Container ports mapped to your computer.                                                   |
| **Names**        | The container's name. You can assign a custom name or let Docker create one automatically. |

If you want to see all containers, including stopped ones, add the `-a` parameter:

```bash
docker container ls -a
```

This shows containers in any state: running, exited, or paused.

If you only want the container IDs, use the `-q` parameter:

```bash
docker container ls -q
```

This is useful for commands that work with multiple containers. For example, to remove all containers at once:

```bash
docker container rm -f $(docker container ls -a -q)
```

You can always check the help for the `ls` command:

```bash
docker container ls -h
```

## Stopping and Starting Containers

To run a container in the background (detached mode):

```bash
docker container run -d --name trivia fundamentalsofdocker/trivia:ed2
```

To stop it:

```bash
docker container stop trivia  # or use the container ID
```

Stopping may take a few seconds. Here’s why:

- Docker sends a `SIGTERM` signal to the main process inside the container.
- If the process doesn’t stop, Docker waits 10 seconds and sends `SIGKILL`, which forcefully stops it.

Stopped containers still occupy space. To remove them:

```bash
docker container rm <container ID>
docker container rm <container name>
```

If a container is still running and you want to remove it, add the force parameter:

```bash
docker container rm -f <container name or ID>
```

This ensures the container is removed no matter its state.


You can find the container ID for more advanced commands:

```bash
export CONTAINER_ID=$(docker container ls -a | grep trivia | awk '{print $1}')
```

For example:
```bash
docker container stop $CONTAINER_ID
docker container rm $CONTAINER_ID
```

This stops and removes the container without typing the long ID manually.

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Running a Docker Container](./2_Running_a_Docker_Container.md)
- [Inspecting containers](./4_Inspecting_containers.md)