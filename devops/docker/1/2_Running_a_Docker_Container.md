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



---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Container architecture](./1_Container_architecture.md)