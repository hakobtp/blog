# Container architecture

Now let’s look at how a system that runs Docker containers is designed at a high level.
The diagram below shows what a computer with Docker installed (a Docker host) looks like:

<p align="center">
    <img src="./assets/img1.png" alt="img1" width="400"/>
</p>

A Docker host is usually built from three main parts:

- The Linux operating system at the bottom
- The container runtime in the middle
- The Docker engine at the top

Docker containers work because the Linux operating system provides special features that make process isolation possible. 
These include **namespaces**, **control groups (cgroups)**, and **layering capabilities**. The container runtime and the Docker 
engine use these features together to create secure, lightweight containers.

Linux namespaces—such as PID namespaces (for process IDs) and network namespaces—separate a container’s processes and network from the rest of the system. This makes each container feel like its own small computer.

Control groups protect the system from “noisy neighbors.” This is a situation where one container uses too many resources and slows down everything else. With cgroups, Docker can limit how much CPU, memory, or other resources each container can use.

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)