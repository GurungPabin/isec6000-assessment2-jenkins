# Start with the official Jenkins long-term support image.
FROM jenkins/jenkins:2.568.3-jdk21

# Administrator access is needed during software installation.
USER root

# Install the Docker client from Docker's official Debian repository.
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    . /etc/os-release && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian ${VERSION_CODENAME} stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends docker-ce-cli && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Run Jenkins as its dedicated non-root user.
USER jenkins

# Add pipeline, Git, Docker agent and access-control support.
RUN jenkins-plugin-cli --plugins \
    workflow-aggregator \
    git \
    docker-workflow \
    matrix-auth \
    credentials-binding \
    junit \
    timestamper \
    pipeline-graph-view
