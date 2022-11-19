FROM fedora:latest

RUN dnf install -y helm unzip openssl

RUN curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.7/2022-10-31/bin/linux/arm64/kubectl
RUN install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

RUN curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_arm64.tar.gz" | tar xz -C /tmp
RUN install -o root -g root -m 0755 /tmp/eksctl /usr/local/bin/eksctl

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install