1. Use this command:
```bash
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
```

2. To join Worker Node, get node token from:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

3. Install and join Worker Node:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.69:6443 K3S_TOKEN=K1002ab209ec0ebdc02a996cab7f3c4ae0eeea1f6ca496d811f178466723d99eb55::server:8c115fae5c7bb1b5fccf03de6b314f4d sh -
```

4. Set role for worker node:
```bash
kubectl label node <node_name> node-role.kubernetes.io/worker=worker
```

5. Install Portainer, access gui using [hostname:30777](http://darktang3nt:30777):
```bash
sudo kubectl apply -n portainer -f https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml
```