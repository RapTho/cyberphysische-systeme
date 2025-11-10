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
├── generatePasswordFile.py
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

The [mosquitto.conf](./files/mosquitto.conf) configures how the broker works. More mosquitto configuration details be found [here](https://mosquitto.org/man/mosquitto-conf-5.html)

### Step 1: Configure access and credentials

- Edit [acl.txt](./files/acl.txt) to define topic access controls.
- Use [generatePasswordFile.py](./files/generatePasswords.py) to generate a `passwords.txt` file for authentication. Feel free to modify the users_and_passwords object to your liking.

Run the script:

```
python generatePasswordFile.py
```

After running the script, your folder structure should include `passwords.txt`:

```
mosquitto
├── Containerfile
├── acl.txt
├── generatePasswordFile.py
├── mosquitto.conf
├── passwords.txt
```

### Step 2: Build the container image

Use the following command to build your container image:

```
podman build --platform linux/amd64 -t mosquitto-${USER}:1.0 --layers=false /path/to/Containerfile
```

- `--platform linux/amd64` specifies the target architecture as x86-64
- `-t` tags the image with a name and version
- `--layers=false` avoids caching intermediate images during the build process

Replace `/path/to/Containerfile` with the actual path to your `mosquitto` folder.

### OPTIONAL: Test locally

To test your image locally, run:

```
podman run -d -v ./:/home/mosquitto/passwords:ro \
              -v ./:/home/mosquitto/acl:ro \
              -v ./:/home/mosquitto/config:ro \
              -p 1883:1883 -p 8083:8083 --name mosquitto mosquitto-${USER}:1.0
```

- `-d` starts the container in the background. Use `podman ps` to display running containers. Add the `-a` option to also display stopped and crashed containers.
- The above command mounts the current directory into the container so it can read `passwords.txt`, `acl.txt`, and `mosquitto.conf`.
- The `-p 1883:1883` and `-p 8083:8083` flag mapps host ports to the container ports where `<hostPort>:<containerPort>`.

### Running on Windows (PowerShell)

Use backticks (\`) for line continuation and double quotes for paths:

```powershell
podman run -d `
  -v "${PWD}:/home/mosquitto/passwords:ro" `
  -v "${PWD}:/home/mosquitto/acl:ro" `
  -v "${PWD}:/home/mosquitto/config:ro" `
  -p 1883:1883 `
  -p 8083:8083 `
  --name mosquitto `
  mosquitto-$env:USERNAME:1.0
```

> `${PWD}` is PowerShell's way to get the current directory path.
>
> **Note:** For more details on Windows-specific commands and environment variables, see the [Windows Environment Setup Guide](./8_Windows-Environment-Setup.md).

With this setup, you’ll have a fully functional, portable mosquitto broker container, secured and ready for use in development environments.
