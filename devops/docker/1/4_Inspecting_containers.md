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


---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Managing Containers](./3_Managing_Containers.md)