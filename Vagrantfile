# -- mode: ruby --
# vi: set ft=ruby :
require 'yaml'
yaml = YAML.load_file("machines.yml")

K8S_IP="192.168.66.20"
K8S_VERSION="1.11.9-00"
DOCKER_VERSION="17.12.1~ce-0~ubuntu"
MYSQL_PASSWORD='devops@integra'


Vagrant.configure("2") do |config|
  yaml.each do |server|
    config.vm.define server["name"] do |srv|
      srv.vm.box = server["sistema"]
      srv.vm.network "private_network", ip: server["ip"]
      srv.vm.hostname = server["hostname"]
      srv.vm.provider "virtualbox" do |vb|
        vb.name = server["name"]
        vb.memory = server["memory"]
        vb.cpus = server["cpus"]
      end

config.vm.provision "shell", inline: <<-SHELL

apt-get update
apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common

if [ $HOSTNAME = "orchestrator" ]; then

 curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
 add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 apt-get update -y
 apt-get install -y docker-ce=#{DOCKER_VERSION}

 curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
 echo 'deb https://apt.kubernetes.io/ kubernetes-xenial main' > /etc/apt/sources.list.d/kubernetes.list
 apt-get update -y
 apt-get install -y kubeadm=#{K8S_VERSION} kubelet=#{K8S_VERSION} kubectl=#{K8S_VERSION}  kubernetes-cni=0.7.5-00

 swapoff -a

 echo "KUBELET_EXTRA_ARGS='--node-ip=#{K8S_IP}'" > /etc/default/kubelet
 kubeadm init --apiserver-advertise-address "#{K8S_IP}" --pod-network-cidr=10.244.0.0/16
 mkdir $HOME/.kube && cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

 #kubectl taint node $HOSTNAME node-role.kubernetes.io/master:NoSchedule
 kubectl taint nodes --all node-role.kubernetes.io/master

fi;

if [ $HOSTNAME = "devops" ]; then
  echo ==== Installing Requirements ==============================================
  sudo apt-get install openjdk-8-jdk-headless -y
  sudo apt-get install -y curl openssh-server ca-certificates

  echo ==== Installing GitLab CE =================================================
  curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
  sudo apt-get install -y gitlab-ce
  sudo gitlab-ctl reconfigure
  sudo gitlab-ctl status

  echo ==== Installing GitLab Multi Runner =======================================
  curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash
  sudo apt-get install -y gitlab-ci-multi-runner

fi;

if [ $HOSTNAME = "mysql" ]; then

 echo "mysql-server mysql-server/root_password password #{MYSQL_PASSWORD}" | sudo debconf-set-selections
 echo "mysql-server mysql-server/root_password_again password #{MYSQL_PASSWORD}" | sudo debconf-set-selections
 sudo apt-get -y install mysql-server
 sudo service mysqld restart


 echo ==== Password Dafault ==============================================

 MYSQL_TEMP_PWD=`sudo cat /var/log/mysqld.log | grep 'A temporary password is generated' | awk -F'root@localhost: ' '{print $2}'`
 echo $MYSQL_TEMP_PWD

 echo ==== Alter Password ==============================================
# mysqladmin -u root -p`echo $MYSQL_TEMP_PWD` password 'Passw0rd!'
 sudo /usr/bin/mysqladmin -u root password '#{MYSQL_PASSWORD}'

 echo ==== Alter Access ==============================================
 mysql -u root -p'#{MYSQL_PASSWORD}' -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '#{MYSQL_PASSWORD}' WITH GRANT OPTION;"
 mysql -u root -p'#{MYSQL_PASSWORD}' -e "DROP USER ''@'localhost';"
 mysql -u root -p'#{MYSQL_PASSWORD}' -e "DROP USER ''@'$(hostname)';"
 mysql -u root -p'#{MYSQL_PASSWORD}' -e "FLUSH PRIVILEGES;"

 echo ==== Restart Services  ==============================================
 sudo service mysqld restart
 sudo chkconfig mysqld on

fi;

 apt-get update -y

  SHELL

  end

 end

end
