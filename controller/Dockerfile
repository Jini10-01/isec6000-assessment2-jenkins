# syntax=docker/dockerfile:1

# Use the official Jenkins Long-Term Support image with Java 21.
FROM jenkins/jenkins:lts-jdk21

# Root permission is required only while installing system packages.
USER root

# Install tools required to configure Docker's official APT repository.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
    \
    # Create a secure directory for the Docker repository signing key.
    && install -m 0755 -d /etc/apt/keyrings \
    \
    # Download Docker's official GPG public key.
    && curl -fsSL https://download.docker.com/linux/debian/gpg \
        -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    \
    # Register Docker's official Debian package repository.
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
        > /etc/apt/sources.list.d/docker.list \
    \
    # Install the Docker client and Buildx plugin.
    # The Docker daemon itself will run in the separate DinD service.
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        docker-ce-cli \
        docker-buildx-plugin \
    \
    # Remove package lists to reduce the final image size.
    && rm -rf /var/lib/apt/lists/*

# Run subsequent commands and the Jenkins service as the built-in
# non-root Jenkins user.
USER jenkins

# Install Jenkins plugins required by the assessment pipeline.
RUN jenkins-plugin-cli --plugins \
    workflow-aggregator \
    git \
    docker-workflow \
    credentials-binding \
    matrix-auth \
    junit
