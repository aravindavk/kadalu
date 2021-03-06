FROM kadalu/builder:latest as builder

ENV PATH="/kadalu/bin:/opt/bin:/opt/sbin:$PATH"

RUN python3 -m pip install kubernetes==11.0.0

FROM ubuntu:20.04 as prod

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update -yq && \
    apt-get install -y --no-install-recommends python3 && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="/kadalu/bin:$PATH"

# kubectl binary
COPY --from=builder /usr/bin/kubectl /usr/bin/kubectl

# actual application to be copied here
# using already installed packages from builder for faster build time
COPY --from=builder /kadalu /kadalu

RUN mkdir -p /kadalu/manifests

COPY templates/services.yaml.j2             /kadalu/templates/services.yaml.j2
COPY templates/server.yaml.j2               /kadalu/templates/server.yaml.j2
COPY templates/csi.yaml.j2                  /kadalu/templates/csi.yaml.j2
COPY templates/csi-driver-crd.yaml.j2       /kadalu/templates/csi-driver-crd.yaml.j2
COPY templates/csi-driver-object.yaml.j2    /kadalu/templates/csi-driver-object.yaml.j2
COPY templates/configmap.yaml.j2            /kadalu/templates/configmap.yaml.j2
COPY templates/storageclass-kadalu.yaml.j2  /kadalu/templates/storageclass-kadalu.yaml.j2
COPY templates/storageclass-kadalu.replica1.yaml.j2  /kadalu/templates/storageclass-kadalu.replica1.yaml.j2
COPY templates/storageclass-kadalu.replica2.yaml.j2  /kadalu/templates/storageclass-kadalu.replica2.yaml.j2
COPY templates/storageclass-kadalu.replica3.yaml.j2  /kadalu/templates/storageclass-kadalu.replica3.yaml.j2
COPY templates/external-storageclass.yaml.j2         /kadalu/templates/external-storageclass.yaml.j2
COPY lib/kadalulib.py                       /kadalu/kadalulib.py
COPY cli/kubectl_kadalu/utils.py            /kadalu/utils.py
COPY operator/main.py                       /kadalu/
COPY operator/start.py                      /kadalu/
COPY cli/build/kubectl-kadalu               /usr/bin/kubectl-kadalu
COPY lib/startup.sh                         /kadalu/startup.sh

RUN chmod +x /kadalu/startup.sh

ARG version="(unknown)"
# Container build time (date -u '+%Y-%m-%dT%H:%M:%S.%NZ')
ARG builddate="(unknown)"

LABEL build-date="${builddate}"
LABEL io.k8s.description="KaDalu Operator"
LABEL name="kadalu-operator"
LABEL Summary="KaDalu Operator"
LABEL vcs-type="git"
LABEL vcs-url="https://github.com/kadalu/kadalu"
LABEL vendor="kadalu"
LABEL version="${version}"

ENTRYPOINT ["python3", "/kadalu/start.py"]

# Debugging, Comment the above line and
# uncomment below line
# ENTRYPOINT ["tail", "-f", "/dev/null"]
