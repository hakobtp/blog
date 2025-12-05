# Inspecting containers

A container is a running instance of an image. Every container has its own data, settings, and state.
To see this information, we can use the inspect command. We only need to give Docker the name or the ID of the container we want to check.

For example:

```bash
docker container inspect trivia
```

The command returns a large JSON document with many details about the container.
A shortened example looks like this:

```json
[
    {
        "Id": "326fac64e8...",
        ...
        "State": {
            "Status": "running",
            "Running": true,
            ...
        },
        "Image": "sha256:0bb60b69...",
        ...
        "Mounts": [],
        "Config": {
            "Hostname": "326fac64e822a",
            "Domainname": "",
            ...
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "dddba8285f252b615a42dc...",
            ...    
        },
        ...
    }
]
```

This output contains many useful sections. For example, you can find:

- The container’s unique ID
- The time and date when the container was created
- The image that was used to start the container

Some parts of the output, such as `Mounts` and `NetworkSettings`, may not make sense yet.
We will cover these topics later. For now, think of this whole document as the metadata of the container.

You will use the inspect command often when you need detailed information.


Sometimes we only need one small part of the JSON. Instead of reading the whole document, we can filter it. 
One way is to install `jq`, a tool that helps you work with JSON on the command line:

```bash
sudo apt install -y jq
```

Then we can filter only the container’s state:

```bash
docker container inspect -f "{{json .State}}" trivia_app | jq .
```

This shows only the `.State` section in a clean and readable format.

## Exec into a running container

We may sometimes want to run a command inside a container that is already running.
This is useful for debugging or checking what is happening inside the container.

We only need the container name or ID, and then we choose the command we want to run.

For example, here we open a shell inside a running container:

```bash
docker container exec -i -t trivia /bin/sh
```

- `-i` means interactive mode
- `-t` gives us a terminal interface

After running the command, you’ll see a new prompt, for example:

```bash
/app #
```

This means you are now inside the container. You can run commands such as:

```bash
/app # ps
```

This shows all running processes inside the container.

To leave the shell, press `Ctrl + D`.

You can also run a command without entering interactive mode:

```bash
docker container exec trivia ps
```

Or run a command with environment variables:

```bash
docker container exec -it \
  -e USER_MESSAGE="Hello from inside!" \
  trivia /bin/sh

```

Inside the container:

```bash
/app # echo $USER_MESSAGE
Hello from inside!
```

## Attaching to a running container

The attach command connects your terminal to a container’s standard input and output.
This allows you to watch the container’s live logs or interact with its main process.

```bash
docker container attach trivia
```

You will now see whatever the container prints to the console—for example, log messages that appear every few seconds.

To detach without stopping the container:
- Press `Ctrl + P`, then `Ctrl + Q`

To detach and stop the container:

- Press `Ctrl + C`

### Example with Nginx

Let’s start a separate Nginx container:

```bash
docker run -d --name webdemo -p 8080:80 nginx:alpine

```

Attach your terminal to it:

```bash
docker container attach webdemo
```

You will not see anything at first.
Now open another terminal and send a few requests:

```bash
for i in {1..10}; do curl -4 localhost:8080; done

```

In the attached terminal, you will now see Nginx logs showing each incoming request.

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Managing Containers](./3_Managing_Containers.md)