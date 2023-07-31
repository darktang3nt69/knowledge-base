docker run -d -p 22:22 -p 80:80 -p 443:443  --name gitlab --hostname gitlab.example.com --restart unless-stopped  --shm-size 256m  -v gitlab_config:/etc/gitlab -v gitlab_logs:/var/log/gitlab  -v gitlab_data:/var/opt/gitlab gitlab/gitlab-ce:14.7.0-ce.0

docker run -d -p 9001:9001  --name portainer_agent  --restart=always  -v /var/run/docker.sock:/var/run/docker.sock  -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent:2.18.3

[In this lab exercise you will use below hosts. Please note down some details about these hosts as given below :



student-node :- This host will act as an Ansible master node where you will create playbooks, inventory, roles etc and you will be running your playbooks from this host itself.


node01 :- This host will act as an Ansible client/remote host where you will setup/install some stuff using Ansible playbooks. Below are the SSH credentials for this host:


User: bob
Password: caleston123


node02 :- This host will also act as an Ansible client/remote host where you will setup/install some stuff using Ansible playbooks. Below are the SSH credentials for this host:


User: bob
Password: caleston123


Note: Please type exit or logout on the terminal or press CTRL + d to log out from a specific node.]


apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.27.0-00 && apt-mark hold kubeadm

apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.27.0-00 kubectl=1.27.0-00 && apt-mark hold kubelet kubectl

apiVersion: v1
kind: PersistentVolume
metadata:
 name: pv-log
spec:
 persistentVolumeReclaimPolicy: Retain
 accessModes:
   - ReadWriteMany
 capacity:
   storage: 100Mi
 hostPath:
   path: /pv/log