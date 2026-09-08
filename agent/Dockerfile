# Official Jenkins inbound agent with Java 21.
FROM jenkins/inbound-agent:jdk21

# Root is used only to install the required software.
USER root

# Install Git for checkout and the Docker client for TLS access to DinD.
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl git && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg \
      -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    . /etc/os-release && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian ${VERSION_CODENAME} stable" \
      > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends docker-ce-cli && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Run the agent as the dedicated non-root Jenkins user.
USER jenkins

WORKDIR /home/jenkins/agent
