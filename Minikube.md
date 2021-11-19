## Create Minikube with Vagrant
```
# Minikube Env Vagrantfile 
$samplescript = <<SCRIPT
yum install -y epel-release
yum install -y htop wget
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
wget minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
mv minikube /usr/bin
wget https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl    && chmod +x kubectl
mv kubectl /usr/bin
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
SCRIPT

###
Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "minikube"
  config.vm.network "private_network", ip: "172.31.11.10"

  config.vm.provision "shell", inline: $samplescript

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4086"
    vb.cpus = "2"
  end
end
###
```
