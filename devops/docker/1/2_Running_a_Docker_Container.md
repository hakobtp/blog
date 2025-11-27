# Running a Docker Container

First, let's check your Docker version to make sure everything is set up correctly:

```bash
docker version
```
You should see information about the client and server, like this:

```text
Client:
 Version: 28.4.0
 OS/Arch: windows/amd64

Server:
 Version: 28.4.0
 OS/Arch: linux/amd64
```

This shows that Docker is installed and ready.

Now, let's try running a simple container. Type this command in your terminal:

```bash
docker container run alpine echo "Hello World"
```

The first time you run it, you might see something like this:

```text
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
Digest: sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412
Status: Downloaded newer image for alpine:latest
Hello World
```

- `Unable to find image` means Docker doesn't have this image on your computer yet.
- Docker will download it from the internet (Docker Hub) automatically.
- Finally, it runs the `echo "Hello World"` command inside the container.

If you run the same command again you will only see:

```bash
docker container run alpine echo "Hello World"

Hello World
```

This is because Docker now uses the image stored locally, so no download is needed.

## Understanding the Command

The command we typed has several parts:

```text
docker          container       run          alpine       echo "Hello World"
 |                 |             |             |                 |
Docker CLI       context       action        image        command inside container
```

1. **docker** – the Docker program we are using.
2. **container** – tells Docker we want to work with containers.
3. **run** – the action we want: start a new container.
4. **alpine** – the container image we want to use.
5. **echo "Hello World"** – the command that runs inside the container.

Let's use a different image, like ubuntu, and run a small task:

```bash
docker container run ubuntu uname -a
```

Output might look like this:

```text
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
Digest: sha256:c35e29c9450151419d9448b0fd75374fec4fff364a27f176fb458d472dfc9e54
Status: Downloaded newer image for ubuntu:latest
Linux 19b1c02e6d4e 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun  5 18:30:46 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

Here:

- **ubuntu** is the image.
- **uname -a** runs a command inside the container that shows system information.

Key Points About Images and Tags

- Each container image has a version, called a tag.
- If you don’t specify a tag, Docker uses latest by default.
- The first time you use an image, Docker downloads it. Later runs are faster because the image is stored locally.

Sometimes you want a container to run without showing output in the terminal. Use:

```bash
docker container run -d --name my_ubuntu ubuntu sleep 60
```

- `-d`: run in the background (detached mode).
- `--name`: give the container a specific name (`my_ubuntu`).

Check the container status:

```bash
docker container ls -l   # shows last container status
docker rm -f my_ubuntu   # stops and removes the container
```

This setup lets you experiment with different images and commands easily, and is the basis for learning Docker containers.

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Container architecture](./1_Container_architecture.md)