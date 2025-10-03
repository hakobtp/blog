## Aligning Architecture and Team Organization

```info
Author      Ter-Petrosyan Hakob
```

---

Imagine SoundWave, an online store that sells digital music and physical albums. Currently, SoundWave uses a classic three-layer architecture: a web interface, a backend server handling business logic, and a database storing all the data. Each layer is managed by a separate team: the frontend team manages the website, the backend team manages the server, and the database team handles storage.

<p align="center">
    <img src="./assets/img2.png" alt="img2" width="100"/>
</p>

Now, SoundWave wants to add a simple feature: letting users choose their favorite music genre. On paper, this seems easy—but in our current setup, it touches all three layers. The web interface must show the genre options, the backend must process changes, and the database must store the new information. Each team must update their part and deploy it in the correct order. Even a small change becomes complex and slow because work is spread across multiple teams.

### Why Three-Layer Architectures Are Common

This kind of architecture is not bad—it's just optimized for a different goal. Traditionally, IT teams are grouped by skill. For example:

- Database administrators work only with databases.
- Backend developers focus on server logic.
- Frontend developers handle user interfaces.

This setup leads naturally to three-layered systems, because each layer matches a team’s skills.

This idea reflects a famous principle called Conway’s Law:

> Organizations design systems that mirror their communication structures.

In other words, if your teams are separated by skill, your system will likely have separate layers.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Key Ideas Behind Microservices](./2_Key_Ideas_Behind_Microservices.md)