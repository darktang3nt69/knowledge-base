- `--dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `--dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
- `-o yaml`: This will output the resource definition in YAML format on screen.

#### POD
1. Create an NGINX Pod:
    ```bash
    kubectl run nginx --image=nginx
    ```

2. Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
    ```bash
    kubectl run nginx --image=nginx --dry-run=client -o yaml
    ```

3. Set image of a pod:
    ```bash
    k set image resource/resource_name container_name=new_container
    ```
4. Assign a pod to Node, simplest way is to just add the following to .spec of pod :
    ```yaml
    nodeName: node-name
    ```
    
#### Deployment

1. Create a deployment
    ```bash
    kubectl create deployment --image=nginx nginx
    ```


2. Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
    ```bash
    kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
    ```
 
3. Generate Deployment with 4 Replicas
    ```bash
    kubectl create deployment nginx --image=nginx --replicas=4
    ```
     - You can also scale a deployment using the `kubectl scale` command.
        ```bash
        kubectl scale deployment nginx --replicas=4
        ```
     - Another way to do this is to save the YAML definition to a file and modify**
        ```bash
         kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
        ```

You can then update the YAML file with the replicas or any other field before creating the deployment.


#### Service

1. Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379`(This will automatically use the pod's labels as selectors)`: 
    ```bash
    kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    ```

    - Another way: This will not use the pods labels as selectors, instead it will assume selectors as **app=redis.** [You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191) So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
      ```bash
     kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
       ```

  
2. Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes. (This will automatically use the pod's labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.): 
    ```bash
    kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
    ```

    
   -  Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.  (This will not use the pods labels as selectors)
       ```bash
       kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
        ```

3. Comm between pods in different namespace:
    ```bash
    service-name.namespace.svc.cluster.local
    ```
#### Selectors and Labels

1. Filter resources using lables:
    ```bash
    kubectl get pods --selector label_name:label_value
    ```
2. Count all resources with the specified labels:
    ```bash
    k get all --selector bu=finance --no-headers | wc -l
    ```
3. Use multiple labels:
    ```bash
    k get pods --selector bu=finance,tier=frontend,env=prod
    ```

4. Add labels to Nodes:
    ```bash
    k label nodes node_name label_key=label-value
    ```

#### Taints and Tolerations

1. Apply a taint to a Node:
    - Taint Effect can take one of the three arguments: 
        1. `NoSchedule`:  Do not schedule any pod which cannot tolerate.
        2. `PreferNoSchedule`:  Node scheduler will try not to schedule pods but it is not guaranteed.
        3. `NoExecute`: No new pods will be scheduled. Existing pods on the node, if any, will be evicted, if they do not tolerate the taint. These may have been scheduled before the taint is applied.
    ```bash
    k taint node node-name key=value:taint-effect
    ```

2. Describe a taint on a node:
    ```bash
    k describe node darktang3ntpi | grep Taints
    ```

3. Remove  a taint. Specify the taintname and - at the end of the name:
    ```bash
    k taint node node_name taint_name-
    ```

#### Editing Pods and Deployments
1. Remember, you CANNOT edit specifications of an existing POD other than the below.
    - spec.containers[*].image
    - spec.initContainers[*].image
    - spec.activeDeadlineSeconds
    - spec.tolerations `(only additions to existing tolerations)`
    - spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)`

2. Run the `kubectl edit pod <pod name>` command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.A copy of the file with your changes is saved in a temporary location as shown above.

    1. You can then delete the existing pod by running the command:
        ```bash
        k delete pod pod_name
        ```

   2.  Then create a new pod with your changes using the temporary file, or use `replace --force` instead of create if the resource is not deleted before applying:
        ```bash
        kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
        ```


#### Static Pods
Static pods have nodename append at the end of the pod name. They also have their ownerReferences set to kind `Node`.

1. Place the static pod manifest in the directory that kubelet is watching:
    1. `/etc/kubernetes/manifests/`
    2. Look for `--config` or `--pod-manifest-path` parameter in kubelet service:
        - If kubelet is running as a daemon:
                        ```bash
                            ps -ef | grep /usr/bin/kubelet
                           ```
    3. Place the pod manifests in the above directory.


#### Logs

1. Use the following command to stream the logs. If the pod contains multiple containers, container name has to be specified. use `-f` to view live logs.
    ```bash
    k logs pods pod_name -f
    ```


#### Rollouts and Updates

1. `UPDATE`: 
    - Using deployment file: 
        ```bash
        k apply -f deployment-file.yaml
        ```
    - Using kubectl( name of the container as defined in the pod spec):
        ```bash
        k set image deployment/deployment-name container_name=new_container_name    
        ```

2. `Status`:
    - Get the current rollout status:
        ```bash
        k rollout status deployment/deployment-name
        ```
    - Get the rollout history of the deployment:
        ```bash
        k rollout history deployment/deployment-name
        ```

3. `Rollback` rollback to the previous rollout :
    ```bash
    k rollout undo deployment/deployment-name
    ```


#### Secrets
Also the way kubernetes handles secrets. Such as:
- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
    

Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks)
#### Switching namespace and context
1. Switch Namespace within the current context:
    ```bash
    k config set-context  --current --namespace=new_namespace
    ```
2. View current context:
    ```bash
    k config current-context 
    ```
3. Switch Context(clusters):
    ```bash
    k config use-context cluster_name
    ```
4. View config:
    ```bash
    k config view
    ```

#### Backing up etcd
1. Replace `cacert`, `cert`, `endpoints`, `key` with the value from etcd process or pod.

    ```bash
    etcdctl snapshot save \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --endpoints=https://127.0.0.1:2379 \
    --key=/etc/kubernetes/pki/etcd/server.key \
     /opt/snapshot-pre-boot.db
    ```





#### Roles and ClusterRoles
1. Get Roles:
    ```bash
    k get roles
    ```
2. Inspect permissions given to another user, `--as` is used to perform the action as the specified user:
    ```bash
    k auth can-i get pods --as dev-user
    ```
3. Check to see if I can do everything in my current namespace (`*` means all):
    ```bash
    kubectl auth can-i '*' '*'
    ```
4. Check to see if I can get the job named "bar" in namespace "foo":
    ```bash
    kubectl auth can-i list jobs.batch/bar -n foo
    ```
5. Create a role named "pod-reader" that allows user to perform "get", "watch" and "list" on pods:
    ```bash
    kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
    ```
6. Create a role binding for `dev-user` with role `dev-user`:
    ```bash
    kubectl create rolebinding dev-user-binding --role=dev-user --user=dev-user
    ```
 7. Get verbose output instead of a simple yes or no to check if a user has a certain permissions, Use:
     ```bash
     kubectl --as user-name create deployment nginx --image=nginx
    ```
8. Create a `ClusterRole
9. 
10. 
11. `, user has access to all nodes:
    ```bash
    kubectl create clusterrole michelle-clusterrole --verb=* --resource=nodes
    ```
9. Create a `ClusterRoleBinding` for the user michelle using the above created `ClusterRole`:
    ```bash
    kubectl create clusterrolebinding michelle-clusterrolebindings --clusterrole=michelle-clusterrole --user=michelle
    ```

#### ServiceAccounts and Token
1. Create  a `ServiceAccount`:
    ```bash
    k create serviceaccount service_account_name
    ```
2. Generate a token for a `ServiceAccount`:
    ```bash
    kubectl create token service_account_name
    ```

#### Private repo(docker) secret
1. Use the following command to create `docker-secret` type secret:
    ```bash
    k create secret docker-registry secret-name --docker-username=replace-me --docker-password=replace-me --docker-email=replace-me@myprivateregistry-name.com --docker-server=replace-me.com:5000
    ```


#### Security Context
1. Check the user inside the container in a pod:
    ```bash
    kubectl exec pod-name -- whoami
     ```
 2. 

#### Persistent Volume
1. Create a simple PV with Access Mode: `ReadWriteMany`, Storage: `100Mi`, Reclaim Policy:`Retain`, Host Path: `/pv/log`:
     ```yaml
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
       ```

2. Create a Persistent Volume Claim with storage request: `50Mi` and Access Mode: `ReadWriteOnce`:
    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: claim-log-1
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 50Mi
    ```

3. Attach a PVC to a pod:
     ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp
    spec:
      containers:
      - name: event-simulator
        image: kodekloud/event-simulator
        env:
        - name: LOG_HANDLERS
          value: file
        volumeMounts:
        - mountPath: /log
          name: log-volume
    
      volumes:
      - name: log-volume
        persistentVolumeClaim:
          claimName: claim-log-1
    ```

4. Create a PVC with storageClassName: 
     ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: local-pvc
    spec:
      accessModes:
      - ReadWriteOnce
      storageClassName: local-storage
      resources:
        requests:
          storage: 500Mi
    ```

5. 

#### **Reference:**
 1. [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
2. [https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)