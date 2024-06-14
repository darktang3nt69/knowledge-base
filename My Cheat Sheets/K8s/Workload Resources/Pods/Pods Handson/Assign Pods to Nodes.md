This page shows how to assign a Kubernetes Pod to a particular node in a Kubernetes cluster.

To check the version, enter `kubectl version`.

## Add a label to a node[](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)

1. List the [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) in your cluster, along with their labels:
    
    ```shell
    kubectl get nodes --show-labels
    ```
    
    The output is similar to this:
    
    ```shell
    NAME      STATUS    ROLES    AGE     VERSION        LABELS
    worker0   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker0
    worker1   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker1
    worker2   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker2
    ```
    
2. Choose one of your nodes, and add a label to it:
    
    ```shell
    kubectl label nodes <your-node-name> disktype=ssd
    ```
    
    where `<your-node-name>` is the name of your chosen node.
    
3. Verify that your chosen node has a `disktype=ssd` label:
    
    ```shell
    kubectl get nodes --show-labels
    ```
    
    The output is similar to this:
    
    ```shell
    NAME      STATUS    ROLES    AGE     VERSION        LABELS
    worker0   Ready     <none>   1d      v1.13.0        ...,disktype=ssd,kubernetes.io/hostname=worker0
    worker1   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker1
    worker2   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker2
    ```
    
    In the preceding output, you can see that the `worker0` node has a `disktype=ssd` label.
    

## Create a pod that gets scheduled to your chosen node[](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#create-a-pod-that-gets-scheduled-to-your-chosen-node)

This pod configuration file describes a pod that has a node selector, `disktype: ssd`. This means that the pod will get scheduled on a node that has a `disktype=ssd` label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

1. Use the configuration file to create a pod that will get scheduled on your chosen node:
    
    ```shell
    kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml
    ```
    
2. Verify that the pod is running on your chosen node:
    
    ```shell
    kubectl get pods --output=wide
    ```
    
    The output is similar to this:
    
    ```shell
    NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
    nginx    1/1       Running   0          13s    10.200.0.4   worker0
    ```
    

## Create a pod that gets scheduled to specific node[](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#create-a-pod-that-gets-scheduled-to-specific-node)

You can also schedule a pod to one specific node via setting `nodeName`.

[`pods/pod-nginx-specific-node.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/pod-nginx-specific-node.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy pods/pod-nginx-specific-node.yaml to clipboard")

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

Use the configuration file to create a pod that will get scheduled on `foo-node` only