## Scaling a Deployment

You can scale a Deployment by using the following command:

```shell
kubectl scale deployment/nginx-deployment --replicas=10
```

The output is similar to this:

```
deployment.apps/nginx-deployment scaled
```

Assuming [horizontal Pod autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) is enabled in your cluster, you can set up an autoscaler for your Deployment and choose the minimum and maximum number of Pods you want to run based on the CPU utilization of your existing Pods.

```shell
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

The output is similar to this:

```
deployment.apps/nginx-deployment scaled
```

### Proportional scaling[](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#proportional-scaling)

RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), the Deployment controller balances the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called _proportional scaling_.

For example, you are running a Deployment with 10 replicas, [maxSurge](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge)=3, and [maxUnavailable](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable)=2.

- Ensure that the 10 replicas in your Deployment are running.
    
    ```shell
    kubectl get deploy
    ```
    
    The output is similar to this:
    
    ```
    NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment     10        10        10           10          50s
    ```
    
- You update to a new image which happens to be unresolvable from inside the cluster.
    
    ```shell
    kubectl set image deployment/nginx-deployment nginx=nginx:sometag
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment image updated
    ```
    
- The image update starts a new rollout with ReplicaSet nginx-deployment-1989198191, but it's blocked due to the `maxUnavailable` requirement that you mentioned above. Check out the rollout status:
    
    ```shell
    kubectl get rs
    ```
    
    The output is similar to this:
    
    ```
    NAME                          DESIRED   CURRENT   READY     AGE
    nginx-deployment-1989198191   5         5         0         9s
    nginx-deployment-618515232    8         8         8         1m
    ```
    
- Then a new scaling request for the Deployment comes along. The autoscaler increments the Deployment replicas to 15. The Deployment controller needs to decide where to add these new 5 replicas. If you weren't using proportional scaling, all 5 of them would be added in the new ReplicaSet. With proportional scaling, you spread the additional replicas across all ReplicaSets. Bigger proportions go to the ReplicaSets with the most replicas and lower proportions go to ReplicaSets with less replicas. Any leftovers are added to the ReplicaSet with the most replicas. ReplicaSets with zero replicas are not scaled up.
    

In our example above, 3 replicas are added to the old ReplicaSet and 2 replicas are added to the new ReplicaSet. The rollout process should eventually move all replicas to the new ReplicaSet, assuming the new replicas become healthy. To confirm this, run:

```shell
kubectl get deploy
```

The output is similar to this:

```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```

The rollout status confirms how the replicas were added to each ReplicaSet.

```shell
kubectl get rs
```

The output is similar to this:

```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```