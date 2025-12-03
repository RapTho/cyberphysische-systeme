# Building a container image

In this section, we'll walk through how to create a custom container image for a local mosquitto broker using Podman. This includes setting up authentication and access control via `passwords.txt`, `acl.txt`, and `mosquitto.conf`, and building a container image. Wherever you see `podman` instructions, you can also use `docker` instead.

## What is a Containerfile?

A `Containerfile` (also known as a `Dockerfile` in Docker contexts) is a script that contains a list of instructions used to build a container image. It defines:

- The **base image** to start from.
- Files and configurations to add or copy.
- System setup steps (e.g., creating directories, setting permissions).
- The **entrypoint or command** to run when the container starts.

Each instruction is executed in sequence, and the result is a lightweight, portable, and reproducible image.

## Build the mosquitto container image

First, prepare your folder structure. It should look like this:

```
mosquitto
├── Containerfile
├── acl.txt
├── generatePasswords.py
├── mosquitto.conf
```

> All the files are provided below

## Containerfile explained

```Dockerfile
FROM eclipse-mosquitto:latest

ARG WORKDIR=/home/mosquitto

WORKDIR $WORKDIR

RUN mkdir ${WORKDIR}/config ${WORKDIR}/acl ${WORKDIR}/passwords

RUN chown -R mosquitto:mosquitto ${WORKDIR}

USER mosquitto

EXPOSE 1883
EXPOSE 8083

CMD ["/usr/sbin/mosquitto", "-c", "/home/mosquitto/config/mosquitto.conf"]
```

### Key instructions

- `FROM eclipse-mosquitto:latest`: Uses the official mosquitto base image.
- `ARG`: Sets a variable that is scoped to the build (not accessible during container runtime)
- `WORKDIR`: Sets the working directory inside the container.
- `RUN mkdir ...`: Creates folders to organize your configs, ACLs, and passwords.
- `USER mosquitto`: Ensures the broker runs as a non-root user.
- `EXPOSE`: Declares which ports are used (1883 for MQTT, 8083 for WebSocket).
- `CMD`: Specifies the command that runs when the container starts, pointing to your custom configuration file.

To simplify first-time use, mosquitto allows unauthenticated access by default, unless other configuration is set. The [mosquitto.conf](./files/mosquitto.conf) configures how the broker works, disabling anonymous access, enabling authentication via password and ACL files, and setting up two network listeners (port 1883 for standard MQTT and port 8083 for WebSocket connections). More mosquitto configuration details be found [here](https://mosquitto.org/man/mosquitto-conf-5.html)

### Step 1: Configure access and credentials

- Edit [acl.txt](./files/acl.txt) to define topic access controls.
- Use [generatePasswords.py](./files/generatePasswords.py) to generate a `passwords.txt` file for authentication. Feel free to modify the users_and_passwords object to your liking.

Run the script:

```
python generatePasswords.py
```

After running the script, your folder structure should include `passwords.txt`:

```
mosquitto
├── Containerfile
├── acl.txt
├── generatePasswords.py
├── mosquitto.conf
├── passwords.txt
```

### Step 2: Build the container image

Use the following command to build your container image:<br />
If your system's username uses capital letters, then choose another image name. The `User` variable can't be used in that case.

**Using Podman:**

```
podman build --platform linux/amd64 -t mosquitto-${USER}:1.0 /path/to/Containerfile
```

**Using Docker (with buildx support):**<br />
Note with `docker` the Containerfile must be called `Dockerfile`. Podman can handle both. <br />
Also, to build cross-platform images, you need to use the [buildx](https://docs.docker.com/reference/cli/docker/buildx/build/) docker plugin.

```
docker buildx build --platform linux/amd64 -t mosquitto-${USER}:1.0 /path/to/Dockerfile
```

- `--platform linux/amd64` emulates the container image's target platform and architecture as linux / x86-64. If you're building on a device with a different architecture (e.g., Apple Silicon with ARM processors), this flag ensures the build process emulates the correct target architecture for deployment on IBM Code Engine.
- `-t` tags the image with a name and version

Replace `/path/to/Containerfile` with the actual path to your `mosquitto` folder.

### OPTIONAL: Test locally

To test your image locally, run:

```
podman run -d --platform linux/amd64 \
              -v ./:/home/mosquitto/passwords:ro \
              -v ./:/home/mosquitto/acl:ro \
              -v ./:/home/mosquitto/config:ro \
              -p 1883:1883 -p 8083:8083 --name mosquitto mosquitto-${USER}:1.0
```

- `-d` starts the container in the background. Use `podman ps` to display running containers. Add the `-a` option to also display stopped and crashed containers.
- The above command mounts the current directory into the container so it can read `passwords.txt`, `acl.txt`, and `mosquitto.conf`.
- The `-p 1883:1883` and `-p 8083:8083` flag mapps host ports to the container ports where `<hostPort>:<containerPort>`. The host port must be free. You cannot map multiple containers to the same host port.

### Running on Windows (PowerShell)

Use backticks (\`) for line continuation and double quotes for paths:

```powershell
podman run -d `
  --platform linux/amd64 `
  -v "${PWD}:/home/mosquitto/passwords:ro" `
  -v "${PWD}:/home/mosquitto/acl:ro" `
  -v "${PWD}:/home/mosquitto/config:ro" `
  -p 1883:1883 `
  -p 8083:8083 `
  --name mosquitto `
  mosquitto-${env:USERNAME}:1.0
```

> `${PWD}` is PowerShell's way to get the current directory path.
>
> **Note:** For more details on Windows-specific commands and environment variables, see the [Windows Environment Setup Guide](./8_Windows-Environment-Setup.md).

With this setup, you’ll have a fully functional, portable mosquitto broker container, secured and ready for use in development environments.
