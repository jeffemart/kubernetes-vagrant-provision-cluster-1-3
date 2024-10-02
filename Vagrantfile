Vagrant.configure("2") do |config|
    (1..1).each do |i|
      config.vm.define "master-#{i}" do |k8s|
        k8s.vm.box = "ubuntu/bionic64"
        k8s.vm.hostname = "master-#{i}"
        k8s.vm.network "private_network", ip: "192.168.56.10#{i}" # Alterado para o intervalo permitido
  
        k8s.ssh.insert_key = false
        k8s.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_rsa']
        k8s.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"
        
        k8s.vm.provider "virtualbox" do |vb|
          vb.gui = false
          vb.cpus = 2
          vb.memory = "2048"
        end

        k8s.vm.provision "shell", inline: <<-SHELL
          # Export mod non interactive
          export DEBIAN_FRONTEND=noninteractive
          export DEBIAN_PRIORITY=critical

          # container runtimes configuration
          # sysctl params required by setup, params persist across reboots
          sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
          # Apply sysctl params without reboot
          sudo sysctl --system
          sudo sysctl net.ipv4.ip_forward

          # containerd configuration
          # Add Docker's official GPG key:
          sudo apt-get update
          sudo apt-get install ca-certificates curl
          sudo install -m 0755 -d /etc/apt/keyrings
          sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
          sudo chmod a+r /etc/apt/keyrings/docker.asc

          # Add the repository to Apt sources:
          echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
          sudo apt-get update

          # To install the latest version
          sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

          cd /etc/containerd/
          sudo containerd config default > config.toml

          # Atualizar a imagem de sandbox para 3.10 no config.toml
          sudo sed -i 's|registry.k8s.io/pause:3.6|registry.k8s.io/pause:3.10|' /etc/containerd/config.toml

          sudo systemctl restart containerd
          sudo systemctl enable containerd

          # kubeadm configuration
          sudo swapoff -a
          sudo apt-get update

          # apt-transport-https may be a dummy package; if so, you can skip that package
          sudo apt-get install -y apt-transport-https ca-certificates curl gpg software-properties-common socat

          # If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
          sudo mkdir -p -m 755 /etc/apt/keyrings
          curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

          # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
          echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

          sudo apt-get update
          sudo apt-get install -y kubelet kubeadm kubectl
          sudo apt-mark hold kubelet kubeadm kubectl

          sudo systemctl enable --now kubelet

          # Master configuration Cluster
          # Inicializar o cluster Kubernetes com o endereço IP privado
          sudo kubeadm init --apiserver-advertise-address=192.168.56.101 --pod-network-cidr=10.244.0.0/16
          
          # Configurar o kubectl para o usuário padrão
          mkdir -p /home/vagrant/.kube
          sudo cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
          sudo chown vagrant:vagrant /home/vagrant/.kube/config
          chmod 600 /home/vagrant/.kube/config
          
          export KUBECONFIG=/home/vagrant/.kube/config

          # Salvar o comando de join em um arquivo
          kubeadm token create --print-join-command > /vagrant/join-command.sh
          chmod +x /vagrant/join-command.sh

          # CNI configuration
          kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
        SHELL
      end
    end
  
    (1..2).each do |i|
      config.vm.define "worker-#{i}" do |k8s|
        k8s.vm.box = "ubuntu/bionic64"
        k8s.vm.hostname = "worker-#{i}"
        k8s.vm.network "private_network", ip: "192.168.56.20#{i}" # Alterado para o intervalo permitido
  
        k8s.ssh.insert_key = false
        k8s.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_rsa']
        k8s.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"

        k8s.vm.provision "shell", inline: <<-SHELL
          # Export mod non interactive
          export DEBIAN_FRONTEND=noninteractive
          
          # container runtimes configuration
          # sysctl params required by setup, params persist across reboots
          sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

          # Apply sysctl params without reboot
          sudo sysctl --system
          sudo sysctl net.ipv4.ip_forward

          # containerd configuration
          # Add Docker's official GPG key:
          sudo apt-get update
          sudo apt-get install ca-certificates curl
          sudo install -m 0755 -d /etc/apt/keyrings
          sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
          sudo chmod a+r /etc/apt/keyrings/docker.asc

          # Add the repository to Apt sources:
          echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
          sudo apt-get update

          # To install the latest version
          sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

          cd /etc/containerd/
          sudo containerd config default > config.toml

          # Atualizar a imagem de sandbox para 3.10 no config.toml
          sudo sed -i 's|registry.k8s.io/pause:3.6|registry.k8s.io/pause:3.10|' /etc/containerd/config.toml

          sudo systemctl restart containerd
          sudo systemctl enable containerd

          # kubeadm configuration
          sudo swapoff -a
          sudo apt-get update

          # apt-transport-https may be a dummy package; if so, you can skip that package
          sudo apt-get install -y apt-transport-https ca-certificates curl gpg software-properties-common socat

          # If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
          sudo mkdir -p -m 755 /etc/apt/keyrings
          curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

          # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
          echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

          sudo apt-get update
          sudo apt-get install -y kubelet kubeadm kubectl
          sudo apt-mark hold kubelet kubeadm kubectl

          sudo systemctl enable --now kubelet

          # Juntar-se ao cluster usando o comando salvo
          bash /vagrant/join-command.sh
        SHELL

        k8s.vm.provider "virtualbox" do |vb|
          vb.gui = false
          vb.cpus = 1
          vb.memory = "2048"
        end
      end
    end
  end