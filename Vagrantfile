# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Use Ubuntu 20.04 as the base VM
  config.vm.box = "bento/ubuntu-20.04"

  # Allocate more resources to the VM
  config.vm.provider "vmware_desktop" do |v|
    v.memory = 4096  # Allocate 4GB of RAM
    v.cpus = 2       # Allocate 2 CPU cores
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"  # Allocate 4GB of RAM
    vb.cpus = 2         # Allocate 2 CPU cores
  end


  # Provisioning script for root user
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    # Update apt package index
    apt-get update

    # Install prerequisites for Docker
    apt-get install -y \
      ca-certificates \
      curl \
      gnupg \
      lsb-release

    # Add Docker's official GPG key
    install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
      gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    chmod a+r /etc/apt/keyrings/docker.gpg

    # Set up the Docker repository
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
      https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | \
      tee /etc/apt/sources.list.d/docker.list > /dev/null

    # Update apt package index again
    apt-get update

    # Install Docker Engine and related packages
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # Start Docker service
    systemctl start docker
    systemctl enable docker

    # Add 'vagrant' user to the 'docker' group
    usermod -aG docker vagrant

    # Install kubectl (ARM64)
    curl -LO "https://dl.k8s.io/release/$(curl -L -s \
      https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl

    # Install Minikube (ARM64)
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64
    install minikube-linux-arm64 /usr/local/bin/minikube
    rm minikube-linux-arm64

    # Install Helm (ARM64)
    curl -LO https://get.helm.sh/helm-v3.13.0-linux-arm64.tar.gz
    tar -zxvf helm-v3.13.0-linux-arm64.tar.gz
    mv linux-arm64/helm /usr/local/bin/helm
    rm -rf helm-v3.13.0-linux-arm64.tar.gz linux-arm64
  SHELL

  # Provisioning script for vagrant user (non-root)
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    # Ensure Docker group membership is applied
    newgrp docker <<EONG

    # Add Datadog Helm repository
    helm repo add datadog https://helm.datadoghq.com
    helm repo update

    # Start Minikube with Docker driver
    minikube start --driver=docker

    # Install Datadog Operator
    helm install datadog-operator datadog/datadog-operator

    # Create Datadog API key secret
    kubectl create secret generic datadog-secret --from-literal=api-key={dd_api_key}

    # Create datadog-agent.yaml in home directory
    cat <<EOF > /home/vagrant/datadog-agent.yaml
apiVersion: "datadoghq.com/v2alpha1"
kind: "DatadogAgent"
metadata:
  name: "datadog"
spec:
  global:
    clusterName: "minikube-1"
    site: "datadoghq.com"
    tags:
      - "env:sandbox"
      - "vm:true"
      - "mac:true"
    credentials:
      apiSecret:
        secretName: "datadog-secret"
        keyName: "api-key"
  features:
    logCollection:
      enabled: true
      containerCollectAll: true
EOF

    # Apply the DatadogAgent manifest
    kubectl apply -f /home/vagrant/datadog-agent.yaml
  SHELL
end