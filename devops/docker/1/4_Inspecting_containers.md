# Inspecting containers

Containers are runtime instances of an image and have a lot of associated data that characterizes their behavior. 
To get more information about a specific container, we can use the inspect command. As usual, we have to provide either the container ID or name to identify the container of which we want to obtain the data. So, let's inspect our sample container:

```bash
docker container inspect trivia
```

The response is a big JSON object full of details. It looks similar to this:

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