# What is Docker?

Docker is an open-source containerization platform that allows developers to package applications and their dependencies into lightweight, portable containers. These containers can run consistently across different environments, making application deployment more efficient and reliable.

- Containers: an isolated environment that runs an application with all its dependencies (libraries, configurations, etc.).
  Containers ensure that your app runs the same way on any machine, whether it's a developer's laptop, a test server, or a cloud service.
- Images: a blueprint for a container including everything needed to run an application (OS, dependencies, code, etc.).
- Docker file: a script that defines how a Docker image is built.
- Docker Hub: a public repository for sharing and downloading pre-built Docker images.
- Docker Compose: a tool for managing multi-container applications.

# Docker file

https://docs.docker.com/reference/dockerfile

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image

## I. ADD vs COPY

### 1️⃣ Format

```dockerfile
  ADD [OPTIONS] <src> ... <dest>
  ADD [OPTIONS] ["<src>", ... "<dest>"]

  COPY [OPTIONS] <src> ... <dest>
  COPY [OPTIONS] ["<src>", ... "<dest>"]
```

### 2️⃣ Behavior

| Feature            | `ADD`                                     | `COPY`                     |
| ------------------ | ----------------------------------------- | -------------------------- |
| **Execution Time** | At image build time                       | At image build time        |
| **Purpose**        | Add local or remote files and directories | Copy files and directories |

### 3️⃣ Use Case

| Scenario                                   | `ADD` | `COPY` |
| ------------------------------------------ | ----- | ------ |
| Copying regular files                      | ✅    | ✅     |
| Copying `.tar.gz` and want auto-extraction | ✅    | ❌     |
| Downloading files from a URL               | ✅    | ❌     |

### ✅ Best Practice:

Prefer `COPY` for most cases because it's straightforward and avoids unintended behavior \
https://docs.docker.com/build/building/best-practices/#add-or-copy

> Why two different instructions that do the same thing? Overall, the greater opinion was that `ADD` was trying to do too much. So the new instruction `COPY` was added into Docker.
>
> > https://www.pluralsight.com/resources/blog/cloud/docker-copy-vs-add-whats-the-difference

## II. CMD vs ENTRYPOINT vs RUN

### 1️⃣ Format

The RUN, CMD, and ENTRYPOINT instructions all have two possible forms:

- Exec form: `INSTRUCTION ["executable","param1","param2"]`
- Shell form: `INSTRUCTION command param1 param2`

### 2️⃣ Behavior

| Feature                     | `CMD`                                                  | `ENTRYPOINT`                                     | `RUN`                           |
| --------------------------- | ------------------------------------------------------ | ------------------------------------------------ | ------------------------------- |
| **Execution Time**          | At container runtime                                   | At container runtime                             | At image build time             |
| **Purpose**                 | Specify default commands                               | Specify default executable.                      | Execute build commands.         |
| **Overriding Behavior**     | Can be overridden using `docker run <image> <command>` | Can be overridden with `docker run --entrypoint` | Cannot be overridden at runtime |
| **Using in docker compose** | `command: ["npm", "run", "dev"]`                       | `entrypoint: ["node", "custom-server.js"]`       | None                            |

### 3️⃣ Use Case

| Scenario                                                            | `CMD` | `ENTRYPOINT`                               | `RUN` |
| ------------------------------------------------------------------- | ----- | ------------------------------------------ | ----- |
| **Setting a default command for a container**                       | ✅    | ❌ (but can be used with `CMD`)            | ❌    |
| **Forcing a command to always run**                                 | ❌    | ✅                                         | ❌    |
| **Executing commands during build (e.g., installing dependencies)** | ❌    | ❌                                         | ✅    |
| **Providing flexibility for users to change commands**              | ✅    | ❌ (unless overridden with `--entrypoint`) | ❌    |

### ✅ Best Practices:

- **`RUN`** (**Most common**): Use for installing dependencies and setting up the container **at build time** (used in almost every Dockerfile for installing dependencies)
- **`CMD`** vs **`ENTRYPOINT`**: (https://docs.docker.com/reference/dockerfile/#understand-how-cmd-and-entrypoint-interact)
  - **`CMD`** (**Very common**): Use for setting a **default command** that users can override (used to define default runtime commands)
  - **`ENTRYPOINT`** (**Less common**): Use when you want a command **to always run** (mainly used when enforcing a required command)
  - Dockerfile should specify at least one of **`CMD`** or **`ENTRYPOINT`** commands
  - The table below shows what command is executed for different **`ENTRYPOINT`** / CMD combinations:

|                              | No `ENTRYPOINT`            | `ENTRYPOINT` exec_entry p1_entry | `ENTRYPOINT` ["exec_entry", "p1_entry"]        |
| ---------------------------- | -------------------------- | -------------------------------- | ---------------------------------------------- |
| No `CMD`                     | error, not allowed         | /bin/sh -c exec_entry p1_entry   | exec_entry p1_entry                            |
| `CMD` ["exec_cmd", "p1_cmd"] | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry   | exec_entry p1_entry exec_cmd p1_cmd            |
| `CMD`D exec_cmd p1_cmd       | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry   | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |
