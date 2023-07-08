docker run -d -p 22:22 -p 80:80 -p 443:443  --name gitlab --hostname gitlab.example.com --restart unless-stopped  --shm-size 256m  -v gitlab_config:/etc/gitlab -v gitlab_logs:/var/log/gitlab  -v gitlab_data:/var/opt/gitlab gitlab/gitlab-ce:14.7.0-ce.0

docker run -d -p 9001:9001  --name portainer_agent  --restart=always  -v /var/run/docker.sock:/var/run/docker.sock  -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent:2.18.3


wWI-3F-CPbBqQ-Qv2DLiWWunBlxJQjRRyVSI1LQe


apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.26.6-00 && apt-mark hold kubeadm

apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.26.6-00 kubectl=1.26.6-00 && apt-mark hold kubelet kubectl