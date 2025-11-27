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

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Running a Docker Container](./2_Running_a_Docker_Container.md)