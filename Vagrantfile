# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Use Ubuntu 20.04 as the base VM
  config.vm.box = "bento/ubuntu-20.04"

  # Enable provisioning with a shell script.
  config.vm.provision "shell", inline: <<-SHELL
    # Update apt package index
    sudo apt-get update

    # Install prerequisites for Docker
    sudo apt-get install -y \
      ca-certificates \
      curl \
      gnupg \
      lsb-release

    # Add Docker's official GPG key
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
      sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Set up the Docker repository
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
      https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # Update apt package index again
    sudo apt-get update

    # Install Docker Engine and related packages
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # Add 'vagrant' user to the 'docker' group
    sudo usermod -aG docker vagrant

    # Install kubectl (ARM64)
    curl -LO "https://dl.k8s.io/release/$(curl -L -s \
      https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl

    # Install Minikube (ARM64)
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64
    sudo install minikube-linux-arm64 /usr/local/bin/minikube
    rm minikube-linux-arm64

    # Install Helm (ARM64)
    curl -LO https://get.helm.sh/helm-v3.13.0-linux-arm64.tar.gz
    tar -zxvf helm-v3.13.0-linux-arm64.tar.gz
    sudo mv linux-arm64/helm /usr/local/bin/helm
    rm -rf helm-v3.13.0-linux-arm64.tar.gz linux-arm64

  SHELL
end