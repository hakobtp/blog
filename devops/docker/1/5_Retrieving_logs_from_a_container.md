# Retrieving logs from a container

When your application runs inside a Docker container, it is best to send log output to `STDOUT` and `STDERR` instead of writing it to a file. 
Docker can automatically collect anything written to these two streams. This makes it easy for users or external systems to read the logs later.

To see the logs of a container, you can use the `docker container logs` command.
For example, to get the logs of a container named `trivia`, run:

```bash
docker container logs trivia
```

This command shows all logs that Docker has stored for this container.

By default, Docker uses the json-file logging driver.
This driver stores logs as JSON in a file on the host machine. 
If Docker is configured to rotate log files (for example, when they grow too large), then:

- `docker container logs` only shows the logs from the current active file, and not from older rotated log files.

So the command does not always show everything from the start of the container’s life.

If you only want to display the latest entries, you can use `--tail`:

```bash
docker container logs --tail 5 trivia
```

This will show only the last five log lines written by the container.

Sometimes you want to watch logs as they are being produced.
You can use the `--follow` (or `-f`) option:

```bash
docker container logs --tail 5 --follow trivia
```

This shows the latest five lines and then continues to print new log lines in real time—similar to the tail `-f` command on Linux.

## Logging drivers

Docker supports several logging mechanisms called logging drivers.
The Docker daemon chooses the default driver, but you can change it for individual containers.

Here are some of the most common drivers:

| Driver        | Description                                                           |
| ------------- | --------------------------------------------------------------------- |
| **none**      | Disables logging for the container. No log output is stored.          |
| **json-file** | The default driver. Logs are stored as JSON files on the host.        |
| **journald**  | Sends logs to the `journald` service, if it is available on the host. |
| **syslog**    | Sends log messages to the system’s `syslog` daemon.                   |
| **gelf**      | Sends logs to a GELF endpoint, such as Graylog or Logstash.           |
| **fluentd**   | Forwards logs to a local `fluentd` service if installed.              |

> **Important:** The `docker container logs` command only works with 
> `json-file` and `journald`. Other drivers do not support reading logs with this command.

### Using a logging driver for a single container

You can set a logging driver for each container individually.
For example, the following command starts a BusyBox container that uses the none driver:

```bash
docker container run --name test -it \
  --log-driver none \
  busybox sh -c 'for N in 1 2 3; do echo "Hello $N"; done'
```

If you now try to read the logs:

```bash
docker container logs test
```

you will get:

```text
Error response from daemon: configured logging driver does not support reading
```

This is expected because the none driver does not store logs at all.

You can remove the container with:

```bash
docker container rm test
```

### Advanced: Changing the default logging driver

You can change the default logging driver for the entire Docker daemon. This can help control how Docker stores logs and prevent log files from growing too large.

**Step 1: Edit the Docker daemon configuration**

On a Linux machine, open the Docker configuration file:

```bash
sudo vi /etc/docker/daemon.json
```

If the file does not exist, you can create it. Add the following content:

```bash
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": 3
    }
}
```

This configuration tells Docker to:

- Use the `json-file` logging driver
- Limit each log file to 10 MB
- Keep 3 rotated log files before deleting the oldest one

This helps prevent your disk from filling up with large log files.

**Step 2: Apply the changes**

To reload the Docker daemon configuration without restarting Docker, run:

```bash
sudo kill -SIGHUP $(pidof dockerd)
```

Docker will now use the new logging settings.


### Why log rotation matters

Without log rotation, logs can grow very large and fill your disk. For example, a web server or API service may write thousands of lines per second. Using max-size and max-file prevents problems caused by full disks.

Example use cases for different drivers

- `json-file` **:** Good for local development and debugging
- `journald` **:** Useful if your host uses systemd and you want centralized logging
- `fluentd` **:** For production systems where logs are collected and analyzed centrally
- `gelf` **:** Works well with Graylog or ELK/EFK stacks

To see which drivers are available on your system and which one is the default, run:

```bash
$ docker info | grep "Logging Driver"
 Logging Driver: json-file


$ docker info | grep "Log"
 Logging Driver: json-file
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
```

This shows your default driver and supported drivers.

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Inspecting containers](./4_Inspecting_containers.md)