FROM quay.io/redhat-user-workloads/rh-openshift-builds-tenant/openshift-jenkins-2-462/jenkinsci-remoting-2-462:fa36efb68e514890faaf0cbc0499a788083407a5 AS remoting

FROM registry.access.redhat.com/ubi9/openjdk-17-runtime:1.20-2.1725851029 AS agent

USER root

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

RUN groupadd -g "${gid}" "${group}" \
  && useradd -l -c "Jenkins user" -d /home/"${user}" -u "${uid}" -g "${gid}" -m "${user}" || echo "user ${user} already exists."

ARG AGENT_WORKDIR=/home/"${user}"/agent
ENV TZ=Etc/UTC

RUN microdnf install --disableplugin=subscription-manager --setopt=install_weak_deps=0 --setopt=tsflags=nodocs -y \
      ca-certificates \
      fontconfig \
      git \
      git-lfs \
      less \
      patch \
      tzdata \
    && microdnf clean --disableplugin=subscription-manager all

ARG VERSION=3261.v9c670a_4748a_9
COPY --from=remoting --chown="${user}":"${group}" /deployments/remoting.jar /usr/share/jenkins/agent.jar
RUN chmod 0644 /usr/share/jenkins/agent.jar \
    && ln -sf /usr/share/jenkins/agent.jar /usr/share/jenkins/slave.jar

COPY ../../jenkins-agent /usr/local/bin/jenkins-agent
RUN chmod +x /usr/local/bin/jenkins-agent && \
    ln -s /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-slave

ENV LANG C.UTF-8

# KFLUXBUGS-1657: mkdir -p fails if run as nonroot
# Adding `chown` commands to ensure proper file ownership after running as root
ENV AGENT_WORKDIR=${AGENT_WORKDIR}
RUN mkdir -p /home/"${user}"/.jenkins && \
    chown "${user}":"${group}" /home/"${user}"/.jenkins && \
    mkdir -p "${AGENT_WORKDIR}" && \
    chown "${user}":"${group}" "${AGENT_WORKDIR}"

USER ${user}
VOLUME /home/"${user}"/.jenkins
VOLUME "${AGENT_WORKDIR}"
WORKDIR /home/"${user}"
ENV USER=${user}

LABEL \
  org.opencontainers.image.vendor="Red Hat" \
  org.opencontainers.image.title="Red Hat Build of Jenkins Inbound Agent Base container image for UBI 9" \
  org.opencontainers.image.description="This is an image for Jenkins agents using TCP or WebSockets to establish inbound connection to the Jenkins controller" \
  org.opencontainers.image.version="${VERSION}" \
  org.opencontainers.image.url="https://www.jenkins.io/" \
  org.opencontainers.image.source="https://github.com/jenkinsci/docker-agent" \
  org.opencontainers.image.licenses="MIT"

ENTRYPOINT ["/usr/local/bin/jenkins-agent"]
