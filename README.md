# ISEC6000 Assessment 2 - Jenkins Infrastructure

This repository contains the containerized Jenkins infrastructure developed for ISEC6000 Secure DevOps Assessment 2.


## Overview

Jenkins is deployed using Docker Compose with a separate Docker-in-Docker (DinD) service. This architecture allows Jenkins pipelines to use Docker without installing the Docker daemon directly inside the Jenkins controller container.

The configuration provides persistent storage, encrypted Docker communication, restricted network access, health checks, and least-privilege user permissions.


## Repository Structure

```text
.
├── controller/
│   └── Dockerfile
├── compose.yaml
└── README.md
```

- `controller/Dockerfile`: Builds the customized Jenkins controller image with the Docker CLI and required Jenkins plugins.
- `compose.yaml`: Defines the Jenkins controller, DinD service, persistent volumes, network, health checks, and security controls.
- `README.md`: Documents the infrastructure, security design, and operating procedures.


## Architecture

The environment contains two main services:

- **Jenkins controller:** Provides the Jenkins web interface and manages pipeline execution.
- **Docker-in-Docker service:** Provides an isolated Docker daemon that Jenkins can use to build and manage Docker images.

Jenkins communicates with the DinD service through the internal Docker Compose network using TLS certificates.


## Security Controls

The following security controls are applied:

- Jenkins runs as its built-in non-root `jenkins` user.
- The Docker daemon runs in a separate DinD container.
- TLS certificates protect communication between Jenkins and Docker.
- Jenkins is bound to `127.0.0.1:8080` and accessed through an SSH tunnel.
- Jenkins user self-registration is disabled.
- Matrix-based security implements least-privilege access control.
- The administrator receives full administrative access.
- The Pipeline User receives only the permissions required to view and trigger jobs.
- Anonymous and general authenticated users receive no permissions.
- The `no-new-privileges` option reduces the risk of privilege escalation.


## Access-Control Configuration

The following permissions are configured in Jenkins:

| User or group | Permissions |
|---|---|
| Administrator | `Overall/Administer` |
| `pipeline_user` | `Overall/Read`, `View/Read`, `Job/Read`, `Job/Build` |
| Anonymous | No permissions |
| Authenticated Users | No general permissions |


## Prerequisites

The host system requires:

- Docker Engine
- Docker Compose
- Git
- SSH access to the RONIN Ubuntu VM

Verify Docker and Docker Compose:

```bash
docker --version
docker compose version
```


## Start the Environment

From the repository directory, build and start the services:

```bash
sudo docker compose up -d --build
```

Check their status:

```bash
sudo docker compose ps
```

The Jenkins and DinD containers should be running, and configured health checks should report a healthy status.


## Verify the Jenkins Runtime User

Confirm that the Jenkins controller runs as the non-root `jenkins` user:

```bash
sudo docker compose exec jenkins id
```

The result should identify the current user as `jenkins`, rather than `root`.


## Verify Docker-in-Docker Connectivity

Confirm that the Jenkins container can securely communicate with the DinD service:

```bash
sudo docker compose exec jenkins docker version
```

A successful result should display information for both the Docker Client and Docker Server.


## Access Jenkins Securely

Jenkins listens only on the loopback interface of the RONIN VM. Create an SSH tunnel from the local computer:

```bash
ssh -i "<PRIVATE_KEY_PATH>" -L 8080:127.0.0.1:8080 ubuntu@<RONIN_SSH_HOST>
```

After establishing the tunnel, open:

```text
http://localhost:8080
```

Keep the SSH terminal open while accessing Jenkins.


## Persistence Verification

Jenkins configuration, users, jobs, and build data are stored in the persistent `jenkins_home` volume.

Recreate the containers without removing their named volumes:

```bash
sudo docker compose down
sudo docker compose up -d
```

After the containers restart, the existing Jenkins users and security settings should remain available.

> Do not use `docker compose down -v` unless all persistent Jenkins data is intentionally being deleted.


## Stop the Environment

Stop and remove the containers and Compose network while retaining persistent volumes:

```bash
sudo docker compose down
```


## Configuration Validation

Validate the Docker Compose configuration:

```bash
sudo docker compose config
```

Check the repository before committing changes:

```bash
git status
git diff --check
```


## References

- [Installing Jenkins with Docker](https://www.jenkins.io/doc/book/installing/docker/)
- [Jenkins Matrix Authorization Strategy](https://plugins.jenkins.io/matrix-auth/)
- [Docker Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker security documentation](https://docs.docker.com/engine/security/)
