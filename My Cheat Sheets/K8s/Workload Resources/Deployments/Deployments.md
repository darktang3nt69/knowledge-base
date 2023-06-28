# 1 - Deployments[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-a2dc0393e0c4079e1c504b6429844e86)

A _Deployment_ provides declarative updates for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

You describe a _desired state_ in a Deployment, and the Deployment [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

**Note:** Do not manage ReplicaSets owned by a Deployment. Consider opening an issue in the main Kubernetes repository if your use case is not covered below.

## Use Case[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#use-case)

The following are typical use cases for Deployments:

- [Create a Deployment to rollout a ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#creating-a-deployment). The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
- [Declare the new state of the Pods](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#updating-a-deployment) by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
- [Rollback to an earlier Deployment revision](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-back-a-deployment) if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
- [Scale up the Deployment to facilitate more load](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#scaling-a-deployment).
- [Pause the rollout of a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pausing-and-resuming-a-deployment) to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
- [Use the status of the Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-status) as an indicator that a rollout has stuck.
- [Clean up older ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#clean-up-policy) that you don't need anymore.

## Creating a Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#creating-a-deployment)

Before creating a Deployment define an [environment variable](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-environment-variable-for-a-container) for a container.

The following is an example of a Deployment. It creates a ReplicaSet to bring up three `nginx` Pods:

[`controllers/nginx-deployment.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/nginx-deployment.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/nginx-deployment.yaml to clipboard")

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

In this example:

- A Deployment named `nginx-deployment` is created, indicated by the `.metadata.name` field. This name will become the basis for the ReplicaSets and Pods which are created later. See [Writing a Deployment Spec](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-deployment-spec) for more details.
    
- The Deployment creates a ReplicaSet that creates three replicated Pods, indicated by the `.spec.replicas` field.
    
- The `.spec.selector` field defines how the created ReplicaSet finds which Pods to manage. In this case, you select a label that is defined in the Pod template (`app: nginx`). However, more sophisticated selection rules are possible, as long as the Pod template itself satisfies the rule.
    
    **Note:** The `.spec.selector.matchLabels` field is a map of {key,value} pairs. A single {key,value} in the `matchLabels` map is equivalent to an element of `matchExpressions`, whose `key` field is "key", the `operator` is "In", and the `values` array contains only "value". All of the requirements, from both `matchLabels` and `matchExpressions`, must be satisfied in order to match.
    
- The `template` field contains the following sub-fields:
    
    - The Pods are labeled `app: nginx`using the `.metadata.labels` field.
    - The Pod template's specification, or `.template.spec` field, indicates that the Pods run one container, `nginx`, which runs the `nginx` [Docker Hub](https://hub.docker.com/) image at version 1.14.2.
    - Create one container and name it `nginx` using the `.spec.template.spec.containers[0].name` field.

Before you begin, make sure your Kubernetes cluster is up and running. Follow the steps given below to create the above Deployment:

1. Create the Deployment by running the following command:
    
    ```shell
    kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
    ```
    
2. Run `kubectl get deployments` to check if the Deployment was created.
    
    If the Deployment is still being created, the output is similar to the following:
    
    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   0/3     0            0           1s
    ```
    
    When you inspect the Deployments in your cluster, the following fields are displayed:
    
    - `NAME` lists the names of the Deployments in the namespace.
    - `READY` displays how many replicas of the application are available to your users. It follows the pattern ready/desired.
    - `UP-TO-DATE` displays the number of replicas that have been updated to achieve the desired state.
    - `AVAILABLE` displays how many replicas of the application are available to your users.
    - `AGE` displays the amount of time that the application has been running.
    
    Notice how the number of desired replicas is 3 according to `.spec.replicas` field.
    
3. To see the Deployment rollout status, run `kubectl rollout status deployment/nginx-deployment`.
    
    The output is similar to:
    
    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    deployment "nginx-deployment" successfully rolled out
    ```
    
4. Run the `kubectl get deployments` again a few seconds later. The output is similar to this:
    
    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           18s
    ```
    
    Notice that the Deployment has created all three replicas, and all replicas are up-to-date (they contain the latest Pod template) and available.
    
5. To see the ReplicaSet (`rs`) created by the Deployment, run `kubectl get rs`. The output is similar to this:
    
    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-75675f5897   3         3         3       18s
    ```
    
    ReplicaSet output shows the following fields:
    
    - `NAME` lists the names of the ReplicaSets in the namespace.
    - `DESIRED` displays the desired number of _replicas_ of the application, which you define when you create the Deployment. This is the _desired state_.
    - `CURRENT` displays how many replicas are currently running.
    - `READY` displays how many replicas of the application are available to your users.
    - `AGE` displays the amount of time that the application has been running.
    
    Notice that the name of the ReplicaSet is always formatted as `[DEPLOYMENT-NAME]-[HASH]`. This name will become the basis for the Pods which are created.
    
    The `HASH` string is the same as the `pod-template-hash` label on the ReplicaSet.
    
6. To see the labels automatically generated for each Pod, run `kubectl get pods --show-labels`. The output is similar to:
    
    ```
    NAME                                READY     STATUS    RESTARTS   AGE       LABELS
    nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=75675f5897
    nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=75675f5897
    nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=75675f5897
    ```
    
    The created ReplicaSet ensures that there are three `nginx` Pods.
    

**Note:**

You must specify an appropriate selector and Pod template labels in a Deployment (in this case, `app: nginx`).

Do not overlap labels or selectors with other controllers (including other Deployments and StatefulSets). Kubernetes doesn't stop you from overlapping, and if multiple controllers have overlapping selectors those controllers might conflict and behave unexpectedly.

### Pod-template-hash label[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-template-hash-label)

**Caution:** Do not change this label.

The `pod-template-hash` label is added by the Deployment controller to every ReplicaSet that a Deployment creates or adopts.

This label ensures that child ReplicaSets of a Deployment do not overlap. It is generated by hashing the `PodTemplate` of the ReplicaSet and using the resulting hash as the label value that is added to the ReplicaSet selector, Pod template labels, and in any existing Pods that the ReplicaSet might have.

## Updating a Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#updating-a-deployment)

**Note:** A Deployment's rollout is triggered if and only if the Deployment's Pod template (that is, `.spec.template`) is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout.

Follow the steps given below to update your Deployment:

1. Let's update the nginx Pods to use the `nginx:1.16.1` image instead of the `nginx:1.14.2` image.
    
    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
    ```
    
    or use the following command:
    
    ```shell
    kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    ```
    
    where `deployment/nginx-deployment` indicates the Deployment, `nginx` indicates the Container the update will take place and `nginx:1.16.1` indicates the new image and its tag.
    
    The output is similar to:
    
    ```
    deployment.apps/nginx-deployment image updated
    ```
    
    Alternatively, you can `edit` the Deployment and change `.spec.template.spec.containers[0].image` from `nginx:1.14.2` to `nginx:1.16.1`:
    
    ```shell
    kubectl edit deployment/nginx-deployment
    ```
    
    The output is similar to:
    
    ```
    deployment.apps/nginx-deployment edited
    ```
    
2. To see the rollout status, run:
    
    ```shell
    kubectl rollout status deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    ```
    
    or
    
    ```
    deployment "nginx-deployment" successfully rolled out
    ```
    

Get more details on your updated Deployment:

- After the rollout succeeds, you can view the Deployment by running `kubectl get deployments`. The output is similar to this:
    
    ```ini
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           36s
    ```
    
- Run `kubectl get rs` to see that the Deployment updated the Pods by creating a new ReplicaSet and scaling it up to 3 replicas, as well as scaling down the old ReplicaSet to 0 replicas.
    
    ```shell
    kubectl get rs
    ```
    
    The output is similar to this:
    
    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       6s
    nginx-deployment-2035384211   0         0         0       36s
    ```
    
- Running `get pods` should now show only the new Pods:
    
    ```shell
    kubectl get pods
    ```
    
    The output is similar to this:
    
    ```
    NAME                                READY     STATUS    RESTARTS   AGE
    nginx-deployment-1564180365-khku8   1/1       Running   0          14s
    nginx-deployment-1564180365-nacti   1/1       Running   0          14s
    nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
    ```
    
    Next time you want to update these Pods, you only need to update the Deployment's Pod template again.
    
    Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).
    
    Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).
    
    For example, if you look at the above Deployment closely, you will see that it first creates a new Pod, then deletes an old Pod, and creates another new one. It does not kill old Pods until a sufficient number of new Pods have come up, and does not create new Pods until a sufficient number of old Pods have been killed. It makes sure that at least 3 Pods are available and that at max 4 Pods in total are available. In case of a Deployment with 4 replicas, the number of Pods would be between 3 and 5.
    
- Get details of your Deployment:
    
    ```shell
    kubectl describe deployments
    ```
    
    The output is similar to this:
    
    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision=2
    Selector:               app=nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
       Containers:
        nginx:
          Image:        nginx:1.16.1
          Port:         80/TCP
          Environment:  <none>
          Mounts:       <none>
        Volumes:        <none>
      Conditions:
        Type           Status  Reason
        ----           ------  ------
        Available      True    MinimumReplicasAvailable
        Progressing    True    NewReplicaSetAvailable
      OldReplicaSets:  <none>
      NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
      Events:
        Type    Reason             Age   From                   Message
        ----    ------             ----  ----                   -------
        Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
        Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
        Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
    ```
    
    Here you see that when you first created the Deployment, it created a ReplicaSet (nginx-deployment-2035384211) and scaled it up to 3 replicas directly. When you updated the Deployment, it created a new ReplicaSet (nginx-deployment-1564180365) and scaled it up to 1 and waited for it to come up. Then it scaled down the old ReplicaSet to 2 and scaled up the new ReplicaSet to 2 so that at least 3 Pods were available and at most 4 Pods were created at all times. It then continued scaling up and down the new and the old ReplicaSet, with the same rolling update strategy. Finally, you'll have 3 available replicas in the new ReplicaSet, and the old ReplicaSet is scaled down to 0.
    

**Note:** Kubernetes doesn't count terminating Pods when calculating the number of `availableReplicas`, which must be between `replicas - maxUnavailable` and `replicas + maxSurge`. As a result, you might notice that there are more Pods than expected during a rollout, and that the total resources consumed by the Deployment is more than `replicas + maxSurge` until the `terminationGracePeriodSeconds` of the terminating Pods expires.

### Rollover (aka multiple updates in-flight)[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rollover-aka-multiple-updates-in-flight)

Each time a new Deployment is observed by the Deployment controller, a ReplicaSet is created to bring up the desired Pods. If the Deployment is updated, the existing ReplicaSet that controls Pods whose labels match `.spec.selector` but whose template does not match `.spec.template` are scaled down. Eventually, the new ReplicaSet is scaled to `.spec.replicas` and all old ReplicaSets is scaled to 0.

If you update a Deployment while an existing rollout is in progress, the Deployment creates a new ReplicaSet as per the update and start scaling that up, and rolls over the ReplicaSet that it was scaling up previously -- it will add it to its list of old ReplicaSets and start scaling it down.

For example, suppose you create a Deployment to create 5 replicas of `nginx:1.14.2`, but then update the Deployment to create 5 replicas of `nginx:1.16.1`, when only 3 replicas of `nginx:1.14.2` had been created. In that case, the Deployment immediately starts killing the 3 `nginx:1.14.2` Pods that it had created, and starts creating `nginx:1.16.1` Pods. It does not wait for the 5 replicas of `nginx:1.14.2` to be created before changing course.

### Label selector updates[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#label-selector-updates)

It is generally discouraged to make label selector updates and it is suggested to plan your selectors up front. In any case, if you need to perform a label selector update, exercise great caution and make sure you have grasped all of the implications.

**Note:** In API version `apps/v1`, a Deployment's label selector is immutable after it gets created.

- Selector additions require the Pod template labels in the Deployment spec to be updated with the new label too, otherwise a validation error is returned. This change is a non-overlapping one, meaning that the new selector does not select ReplicaSets and Pods created with the old selector, resulting in orphaning all old ReplicaSets and creating a new ReplicaSet.
- Selector updates changes the existing value in a selector key -- result in the same behavior as additions.
- Selector removals removes an existing key from the Deployment selector -- do not require any changes in the Pod template labels. Existing ReplicaSets are not orphaned, and a new ReplicaSet is not created, but note that the removed label still exists in any existing Pods and ReplicaSets.

## Rolling Back a Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-back-a-deployment)

Sometimes, you may want to rollback a Deployment; for example, when the Deployment is not stable, such as crash looping. By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want (you can change that by modifying revision history limit).

**Note:** A Deployment's revision is created when a Deployment's rollout is triggered. This means that the new revision is created if and only if the Deployment's Pod template (`.spec.template`) is changed, for example if you update the labels or container images of the template. Other updates, such as scaling the Deployment, do not create a Deployment revision, so that you can facilitate simultaneous manual- or auto-scaling. This means that when you roll back to an earlier revision, only the Deployment's Pod template part is rolled back.

- Suppose that you made a typo while updating the Deployment, by putting the image name as `nginx:1.161` instead of `nginx:1.16.1`:
    
    ```shell
    kubectl set image deployment/nginx-deployment nginx=nginx:1.161
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment image updated
    ```
    
- The rollout gets stuck. You can verify it by checking the rollout status:
    
    ```shell
    kubectl rollout status deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
    ```
    
- Press Ctrl-C to stop the above rollout status watch. For more information on stuck rollouts, [read more here](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-status).
    
- You see that the number of old replicas (`nginx-deployment-1564180365` and `nginx-deployment-2035384211`) is 2, and new replicas (nginx-deployment-3066724191) is 1.
    
    ```shell
    kubectl get rs
    ```
    
    The output is similar to this:
    
    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       25s
    nginx-deployment-2035384211   0         0         0       36s
    nginx-deployment-3066724191   1         1         0       6s
    ```
    
- Looking at the Pods created, you see that 1 Pod created by new ReplicaSet is stuck in an image pull loop.
    
    ```shell
    kubectl get pods
    ```
    
    The output is similar to this:
    
    ```
    NAME                                READY     STATUS             RESTARTS   AGE
    nginx-deployment-1564180365-70iae   1/1       Running            0          25s
    nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
    nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
    nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
    ```
    
    **Note:** The Deployment controller stops the bad rollout automatically, and stops scaling up the new ReplicaSet. This depends on the rollingUpdate parameters (`maxUnavailable` specifically) that you have specified. Kubernetes by default sets the value to 25%.
    
- Get the description of the Deployment:
    
    ```shell
    kubectl describe deployment
    ```
    
    The output is similar to this:
    
    ```
    Name:           nginx-deployment
    Namespace:      default
    CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
    Labels:         app=nginx
    Selector:       app=nginx
    Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
    StrategyType:       RollingUpdate
    MinReadySeconds:    0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.161
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    ReplicaSetUpdated
    OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
    NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
    Events:
      FirstSeen LastSeen    Count   From                    SubObjectPath   Type        Reason              Message
      --------- --------    -----   ----                    -------------   --------    ------              -------
      1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
    ```
    
    To fix this, you need to rollback to a previous revision of Deployment that is stable.
    

### Checking Rollout History of a Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#checking-rollout-history-of-a-deployment)

Follow the steps given below to check the rollout history:

1. First, check the revisions of this Deployment:
    
    ```shell
    kubectl rollout history deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml
    2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
    ```
    
    `CHANGE-CAUSE` is copied from the Deployment annotation `kubernetes.io/change-cause` to its revisions upon creation. You can specify the`CHANGE-CAUSE` message by:
    
    - Annotating the Deployment with `kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"`
    - Manually editing the manifest of the resource.
2. To see the details of each revision, run:
    
    ```shell
    kubectl rollout history deployment/nginx-deployment --revision=2
    ```
    
    The output is similar to this:
    
    ```
    deployments "nginx-deployment" revision 2
      Labels:       app=nginx
              pod-template-hash=1159050644
      Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
      Containers:
       nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
         QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
      No volumes.
    ```
    

### Rolling Back to a Previous Revision[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-back-to-a-previous-revision)

Follow the steps given below to rollback the Deployment from the current version to the previous version, which is version 2.

1. Now you've decided to undo the current rollout and rollback to the previous revision:
    
    ```shell
    kubectl rollout undo deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment rolled back
    ```
    
    Alternatively, you can rollback to a specific revision by specifying it with `--to-revision`:
    
    ```shell
    kubectl rollout undo deployment/nginx-deployment --to-revision=2
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment rolled back
    ```
    
    For more details about rollout related commands, read [`kubectl rollout`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout).
    
    The Deployment is now rolled back to a previous stable revision. As you can see, a `DeploymentRollback` event for rolling back to revision 2 is generated from Deployment controller.
    
2. Check if the rollback was successful and the Deployment is running as expected, run:
    
    ```shell
    kubectl get deployment nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           30m
    ```
    
3. Get the description of the Deployment:
    
    ```shell
    kubectl describe deployment nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision=4
                            kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    Selector:               app=nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
    Events:
      Type    Reason              Age   From                   Message
      ----    ------              ----  ----                   -------
      Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
      Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
      Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
    ```
    

## Scaling a Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#scaling-a-deployment)

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

### Proportional scaling[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#proportional-scaling)

RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), the Deployment controller balances the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called _proportional scaling_.

For example, you are running a Deployment with 10 replicas, [maxSurge](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#max-surge)=3, and [maxUnavailable](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#max-unavailable)=2.

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

## Pausing and Resuming a rollout of a Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pausing-and-resuming-a-deployment)

When you update a Deployment, or plan to, you can pause rollouts for that Deployment before you trigger one or more updates. When you're ready to apply those changes, you resume rollouts for the Deployment. This approach allows you to apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.

- For example, with a Deployment that was created:
    
    Get the Deployment details:
    
    ```shell
    kubectl get deploy
    ```
    
    The output is similar to this:
    
    ```
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     3         3         3            3           1m
    ```
    
    Get the rollout status:
    
    ```shell
    kubectl get rs
    ```
    
    The output is similar to this:
    
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         1m
    ```
    
- Pause by running the following command:
    
    ```shell
    kubectl rollout pause deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment paused
    ```
    
- Then update the image of the Deployment:
    
    ```shell
    kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment image updated
    ```
    
- Notice that no new rollout started:
    
    ```shell
    kubectl rollout history deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```
    
- Get the rollout status to verify that the existing ReplicaSet has not changed:
    
    ```shell
    kubectl get rs
    ```
    
    The output is similar to this:
    
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```
    
- You can make as many updates as you wish, for example, update the resources that will be used:
    
    ```shell
    kubectl set resources deployment/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```
    
    The initial state of the Deployment prior to pausing its rollout will continue its function, but new updates to the Deployment will not have any effect as long as the Deployment rollout is paused.
    
- Eventually, resume the Deployment rollout and observe a new ReplicaSet coming up with all the new updates:
    
    ```shell
    kubectl rollout resume deployment/nginx-deployment
    ```
    
    The output is similar to this:
    
    ```
    deployment.apps/nginx-deployment resumed
    ```
    
- Watch the status of the rollout until it's done.
    
    ```shell
    kubectl get rs -w
    ```
    
    The output is similar to this:
    
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   2         2         2         2m
    nginx-3926361531   2         2         0         6s
    nginx-3926361531   2         2         1         18s
    nginx-2142116321   1         2         2         2m
    nginx-2142116321   1         2         2         2m
    nginx-3926361531   3         2         1         18s
    nginx-3926361531   3         2         1         18s
    nginx-2142116321   1         1         1         2m
    nginx-3926361531   3         3         1         18s
    nginx-3926361531   3         3         2         19s
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         20s
    ```
    
- Get the status of the latest rollout:
    
    ```shell
    kubectl get rs
    ```
    
    The output is similar to this:
    
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```
    

**Note:** You cannot rollback a paused Deployment until you resume it.

## Deployment status[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-status)

A Deployment enters various states during its lifecycle. It can be [progressing](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#progressing-deployment) while rolling out a new ReplicaSet, it can be [complete](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#complete-deployment), or it can [fail to progress](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#failed-deployment).

### Progressing Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#progressing-deployment)

Kubernetes marks a Deployment as _progressing_ when one of the following tasks is performed:

- The Deployment creates a new ReplicaSet.
- The Deployment is scaling up its newest ReplicaSet.
- The Deployment is scaling down its older ReplicaSet(s).
- New Pods become ready or available (ready for at least [MinReadySeconds](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#min-ready-seconds)).

When the rollout becomes “progressing”, the Deployment controller adds a condition with the following attributes to the Deployment's `.status.conditions`:

- `type: Progressing`
- `status: "True"`
- `reason: NewReplicaSetCreated` | `reason: FoundNewReplicaSet` | `reason: ReplicaSetUpdated`

You can monitor the progress for a Deployment by using `kubectl rollout status`.

### Complete Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#complete-deployment)

Kubernetes marks a Deployment as _complete_ when it has the following characteristics:

- All of the replicas associated with the Deployment have been updated to the latest version you've specified, meaning any updates you've requested have been completed.
- All of the replicas associated with the Deployment are available.
- No old replicas for the Deployment are running.

When the rollout becomes “complete”, the Deployment controller sets a condition with the following attributes to the Deployment's `.status.conditions`:

- `type: Progressing`
- `status: "True"`
- `reason: NewReplicaSetAvailable`

This `Progressing` condition will retain a status value of `"True"` until a new rollout is initiated. The condition holds even when availability of replicas changes (which does instead affect the `Available` condition).

You can check if a Deployment has completed by using `kubectl rollout status`. If the rollout completed successfully, `kubectl rollout status` returns a zero exit code.

```shell
kubectl rollout status deployment/nginx-deployment
```

The output is similar to this:

```
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
```

and the exit status from `kubectl rollout` is 0 (success):

```shell
echo $?
```

```
0
```

### Failed Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#failed-deployment)

Your Deployment may get stuck trying to deploy its newest ReplicaSet without ever completing. This can occur due to some of the following factors:

- Insufficient quota
- Readiness probe failures
- Image pull errors
- Insufficient permissions
- Limit ranges
- Application runtime misconfiguration

One way you can detect this condition is to specify a deadline parameter in your Deployment spec: ([`.spec.progressDeadlineSeconds`](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#progress-deadline-seconds)). `.spec.progressDeadlineSeconds` denotes the number of seconds the Deployment controller waits before indicating (in the Deployment status) that the Deployment progress has stalled.

The following `kubectl` command sets the spec with `progressDeadlineSeconds` to make the controller report lack of progress of a rollout for a Deployment after 10 minutes:

```shell
kubectl patch deployment/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

The output is similar to this:

```
deployment.apps/nginx-deployment patched
```

Once the deadline has been exceeded, the Deployment controller adds a DeploymentCondition with the following attributes to the Deployment's `.status.conditions`:

- `type: Progressing`
- `status: "False"`
- `reason: ProgressDeadlineExceeded`

This condition can also fail early and is then set to status value of `"False"` due to reasons as `ReplicaSetCreateError`. Also, the deadline is not taken into account anymore once the Deployment rollout completes.

See the [Kubernetes API conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties) for more information on status conditions.

**Note:** Kubernetes takes no action on a stalled Deployment other than to report a status condition with `reason: ProgressDeadlineExceeded`. Higher level orchestrators can take advantage of it and act accordingly, for example, rollback the Deployment to its previous version.

**Note:** If you pause a Deployment rollout, Kubernetes does not check progress against your specified deadline. You can safely pause a Deployment rollout in the middle of a rollout and resume without triggering the condition for exceeding the deadline.

You may experience transient errors with your Deployments, either due to a low timeout that you have set or due to any other kind of error that can be treated as transient. For example, let's suppose you have insufficient quota. If you describe the Deployment you will notice the following section:

```shell
kubectl describe deployment nginx-deployment
```

The output is similar to this:

```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

If you run `kubectl get deployment nginx-deployment -o yaml`, the Deployment status is similar to this:

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

Eventually, once the Deployment progress deadline is exceeded, Kubernetes updates the status and the reason for the Progressing condition:

```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

You can address an issue of insufficient quota by scaling down your Deployment, by scaling down other controllers you may be running, or by increasing quota in your namespace. If you satisfy the quota conditions and the Deployment controller then completes the Deployment rollout, you'll see the Deployment's status update with a successful condition (`status: "True"` and `reason: NewReplicaSetAvailable`).

```
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```

`type: Available` with `status: "True"` means that your Deployment has minimum availability. Minimum availability is dictated by the parameters specified in the deployment strategy. `type: Progressing` with `status: "True"` means that your Deployment is either in the middle of a rollout and it is progressing or that it has successfully completed its progress and the minimum required new replicas are available (see the Reason of the condition for the particulars - in our case `reason: NewReplicaSetAvailable` means that the Deployment is complete).

You can check if a Deployment has failed to progress by using `kubectl rollout status`. `kubectl rollout status` returns a non-zero exit code if the Deployment has exceeded the progression deadline.

```shell
kubectl rollout status deployment/nginx-deployment
```

The output is similar to this:

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
```

and the exit status from `kubectl rollout` is 1 (indicating an error):

```shell
echo $?
```

```
1
```

### Operating on a failed deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#operating-on-a-failed-deployment)

All actions that apply to a complete Deployment also apply to a failed Deployment. You can scale it up/down, roll back to a previous revision, or even pause it if you need to apply multiple tweaks in the Deployment Pod template.

## Clean up Policy[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#clean-up-policy)

You can set `.spec.revisionHistoryLimit` field in a Deployment to specify how many old ReplicaSets for this Deployment you want to retain. The rest will be garbage-collected in the background. By default, it is 10.

**Note:** Explicitly setting this field to 0, will result in cleaning up all the history of your Deployment thus that Deployment will not be able to roll back.

## Canary Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#canary-deployment)

If you want to roll out releases to a subset of users or servers using the Deployment, you can create multiple Deployments, one for each release, following the canary pattern described in [managing resources](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments).

## Writing a Deployment Spec[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-deployment-spec)

As with all other Kubernetes configs, a Deployment needs `.apiVersion`, `.kind`, and `.metadata` fields. For general information about working with config files, see [deploying applications](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/), configuring containers, and [using kubectl to manage resources](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/) documents.

When the control plane creates new Pods for a Deployment, the `.metadata.name` of the Deployment is part of the basis for naming those Pods. The name of a Deployment must be a valid [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-label-names).

A Deployment also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

### Pod Template[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-template)

The `.spec.template` and `.spec.selector` are the only required fields of the `.spec`.

The `.spec.template` is a [Pod template](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/), except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a Pod template in a Deployment must specify appropriate labels and an appropriate restart policy. For labels, make sure not to overlap with other controllers. See [selector](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#selector).

Only a [`.spec.template.spec.restartPolicy`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Always` is allowed, which is the default if not specified.

### Replicas[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicas)

`.spec.replicas` is an optional field that specifies the number of desired Pods. It defaults to 1.

Should you manually scale a Deployment, example via `kubectl scale deployment deployment --replicas=X`, and then you update that Deployment based on a manifest (for example: by running `kubectl apply -f deployment.yaml`), then applying that manifest overwrites the manual scaling that you previously did.

If a [HorizontalPodAutoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) (or any similar API for horizontal scaling) is managing scaling for a Deployment, don't set `.spec.replicas`.

Instead, allow the Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) to manage the `.spec.replicas` field automatically.

### Selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#selector)

`.spec.selector` is a required field that specifies a [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for the Pods targeted by this Deployment.

`.spec.selector` must match `.spec.template.metadata.labels`, or it will be rejected by the API.

In API version `apps/v1`, `.spec.selector` and `.metadata.labels` do not default to `.spec.template.metadata.labels` if not set. So they must be set explicitly. Also note that `.spec.selector` is immutable after creation of the Deployment in `apps/v1`.

A Deployment may terminate Pods whose labels match the selector if their template is different from `.spec.template` or if the total number of such Pods exceeds `.spec.replicas`. It brings up new Pods with `.spec.template` if the number of Pods is less than the desired number.

**Note:** You should not create other Pods whose labels match this selector, either directly, by creating another Deployment, or by creating another controller such as a ReplicaSet or a ReplicationController. If you do so, the first Deployment thinks that it created these other Pods. Kubernetes does not stop you from doing this.

If you have multiple controllers that have overlapping selectors, the controllers will fight with each other and won't behave correctly.

### Strategy[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#strategy)

`.spec.strategy` specifies the strategy used to replace old Pods by new ones. `.spec.strategy.type` can be "Recreate" or "RollingUpdate". "RollingUpdate" is the default value.

#### Recreate Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#recreate-deployment)

All existing Pods are killed before new ones are created when `.spec.strategy.type==Recreate`.

**Note:** This will only guarantee Pod termination previous to creation for upgrades. If you upgrade a Deployment, all Pods of the old revision will be terminated immediately. Successful removal is awaited before any Pod of the new revision is created. If you manually delete a Pod, the lifecycle is controlled by the ReplicaSet and the replacement will be created immediately (even if the old Pod is still in a Terminating state). If you need an "at most" guarantee for your Pods, you should consider using a [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).

#### Rolling Update Deployment[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-update-deployment)

The Deployment updates Pods in a rolling update fashion when `.spec.strategy.type==RollingUpdate`. You can specify `maxUnavailable` and `maxSurge` to control the rolling update process.

##### Max Unavailable[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#max-unavailable)

`.spec.strategy.rollingUpdate.maxUnavailable` is an optional field that specifies the maximum number of Pods that can be unavailable during the update process. The value can be an absolute number (for example, 5) or a percentage of desired Pods (for example, 10%). The absolute number is calculated from percentage by rounding down. The value cannot be 0 if `.spec.strategy.rollingUpdate.maxSurge` is 0. The default value is 25%.

For example, when this value is set to 30%, the old ReplicaSet can be scaled down to 70% of desired Pods immediately when the rolling update starts. Once new Pods are ready, old ReplicaSet can be scaled down further, followed by scaling up the new ReplicaSet, ensuring that the total number of Pods available at all times during the update is at least 70% of the desired Pods.

##### Max Surge[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#max-surge)

`.spec.strategy.rollingUpdate.maxSurge` is an optional field that specifies the maximum number of Pods that can be created over the desired number of Pods. The value can be an absolute number (for example, 5) or a percentage of desired Pods (for example, 10%). The value cannot be 0 if `MaxUnavailable` is 0. The absolute number is calculated from the percentage by rounding up. The default value is 25%.

For example, when this value is set to 30%, the new ReplicaSet can be scaled up immediately when the rolling update starts, such that the total number of old and new Pods does not exceed 130% of desired Pods. Once old Pods have been killed, the new ReplicaSet can be scaled up further, ensuring that the total number of Pods running at any time during the update is at most 130% of desired Pods.

### Progress Deadline Seconds[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#progress-deadline-seconds)

`.spec.progressDeadlineSeconds` is an optional field that specifies the number of seconds you want to wait for your Deployment to progress before the system reports back that the Deployment has [failed progressing](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#failed-deployment) - surfaced as a condition with `type: Progressing`, `status: "False"`. and `reason: ProgressDeadlineExceeded` in the status of the resource. The Deployment controller will keep retrying the Deployment. This defaults to 600. In the future, once automatic rollback will be implemented, the Deployment controller will roll back a Deployment as soon as it observes such a condition.

If specified, this field needs to be greater than `.spec.minReadySeconds`.

### Min Ready Seconds[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#min-ready-seconds)

`.spec.minReadySeconds` is an optional field that specifies the minimum number of seconds for which a newly created Pod should be ready without any of its containers crashing, for it to be considered available. This defaults to 0 (the Pod will be considered available as soon as it is ready). To learn more about when a Pod is considered ready, see [Container Probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes).

### Revision History Limit[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#revision-history-limit)

A Deployment's revision history is stored in the ReplicaSets it controls.

`.spec.revisionHistoryLimit` is an optional field that specifies the number of old ReplicaSets to retain to allow rollback. These old ReplicaSets consume resources in `etcd` and crowd the output of `kubectl get rs`. The configuration of each Deployment revision is stored in its ReplicaSets; therefore, once an old ReplicaSet is deleted, you lose the ability to rollback to that revision of Deployment. By default, 10 old ReplicaSets will be kept, however its ideal value depends on the frequency and stability of new Deployments.

More specifically, setting this field to zero means that all old ReplicaSets with 0 replicas will be cleaned up. In this case, a new Deployment rollout cannot be undone, since its revision history is cleaned up.

### Paused[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#paused)

`.spec.paused` is an optional boolean field for pausing and resuming a Deployment. The only difference between a paused Deployment and one that is not paused, is that any changes into the PodTemplateSpec of the paused Deployment will not trigger new rollouts as long as it is paused. A Deployment is not paused by default when it is created.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn more about [Pods](https://kubernetes.io/docs/concepts/workloads/pods).
- [Run a stateless application using a Deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/).
- Read the [Deployment](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/) to understand the Deployment API.
- Read about [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) and how you can use it to manage application availability during disruptions.
- Use kubectl to [create a Deployment](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/).

# 2 - ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-d459b930218774655fa7fd1620625539)

A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

## How a ReplicaSet works[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#how-a-replicaset-works)

A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod template.

A ReplicaSet is linked to its Pods via the Pods' [metadata.ownerReferences](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#owners-and-dependents) field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their ownerReferences field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.

A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

## When to use a ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#when-to-use-a-replicaset)

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.

## Example[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#example)

[`controllers/frontend.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/frontend.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/frontend.yaml to clipboard")

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

Saving this manifest into `frontend.yaml` and submitting it to a Kubernetes cluster will create the defined ReplicaSet and the Pods that it manages.

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```

You can then get the current ReplicaSets deployed:

```shell
kubectl get rs
```

And see the frontend one you created:

```
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```

You can also check on the state of the ReplicaSet:

```shell
kubectl describe rs/frontend
```

And you will see output similar to:

```
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"app":"guestbook","tier":"frontend"},"name":"frontend",...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  117s  replicaset-controller  Created pod: frontend-wtsmm
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-b2zdv
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-vcmts
```

And lastly you can check for the Pods brought up:

```shell
kubectl get pods
```

You should see Pod information similar to:

```
NAME             READY   STATUS    RESTARTS   AGE
frontend-b2zdv   1/1     Running   0          6m36s
frontend-vcmts   1/1     Running   0          6m36s
frontend-wtsmm   1/1     Running   0          6m36s
```

You can also verify that the owner reference of these pods is set to the frontend ReplicaSet. To do this, get the yaml of one of the Pods running:

```shell
kubectl get pods frontend-b2zdv -o yaml
```

The output will look similar to this, with the frontend ReplicaSet's info set in the metadata's ownerReferences field:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-02-12T07:06:16Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-b2zdv
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: f391f6db-bb9b-4c09-ae74-6a1f77f3d5cf
...
```

## Non-Template Pod acquisitions[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#non-template-pod-acquisitions)

While you can create bare Pods with no problems, it is strongly recommended to make sure that the bare Pods do not have labels which match the selector of one of your ReplicaSets. The reason for this is because a ReplicaSet is not limited to owning Pods specified by its template-- it can acquire other Pods in the manner specified in the previous sections.

Take the previous frontend ReplicaSet example, and the Pods specified in the following manifest:

[`pods/pod-rs.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/pod-rs.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy pods/pod-rs.yaml to clipboard")

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: hello1
    image: gcr.io/google-samples/hello-app:2.0

---

apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    tier: frontend
spec:
  containers:
  - name: hello2
    image: gcr.io/google-samples/hello-app:1.0
```

As those Pods do not have a Controller (or any object) as their owner reference and match the selector of the frontend ReplicaSet, they will immediately be acquired by it.

Suppose you create the Pods after the frontend ReplicaSet has been deployed and has set up its initial Pod replicas to fulfill its replica count requirement:

```shell
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

The new Pods will be acquired by the ReplicaSet, and then immediately terminated as the ReplicaSet would be over its desired count.

Fetching the Pods:

```shell
kubectl get pods
```

The output shows that the new Pods are either already terminated, or in the process of being terminated:

```
NAME             READY   STATUS        RESTARTS   AGE
frontend-b2zdv   1/1     Running       0          10m
frontend-vcmts   1/1     Running       0          10m
frontend-wtsmm   1/1     Running       0          10m
pod1             0/1     Terminating   0          1s
pod2             0/1     Terminating   0          1s
```

If you create the Pods first:

```shell
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

And then create the ReplicaSet however:

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```

You shall see that the ReplicaSet has acquired the Pods and has only created new ones according to its spec until the number of its new Pods and the original matches its desired count. As fetching the Pods:

```shell
kubectl get pods
```

Will reveal in its output:

```
NAME             READY   STATUS    RESTARTS   AGE
frontend-hmmj2   1/1     Running   0          9s
pod1             1/1     Running   0          36s
pod2             1/1     Running   0          36s
```

In this manner, a ReplicaSet can own a non-homogenous set of Pods

## Writing a ReplicaSet manifest[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-replicaset-manifest)

As with all other Kubernetes API objects, a ReplicaSet needs the `apiVersion`, `kind`, and `metadata` fields. For ReplicaSets, the `kind` is always a ReplicaSet.

When the control plane creates new Pods for a ReplicaSet, the `.metadata.name` of the ReplicaSet is part of the basis for naming those Pods. The name of a ReplicaSet must be a valid [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-label-names).

A ReplicaSet also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

### Pod Template[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-template)

The `.spec.template` is a [pod template](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates) which is also required to have labels in place. In our `frontend.yaml` example we had one label: `tier: frontend`. Be careful not to overlap with the selectors of other controllers, lest they try to adopt this Pod.

For the template's [restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) field, `.spec.template.spec.restartPolicy`, the only allowed value is `Always`, which is the default.

### Pod Selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)

The `.spec.selector` field is a [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/). As discussed [earlier](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#how-a-replicaset-works) these are the labels used to identify potential Pods to acquire. In our `frontend.yaml` example, the selector was:

```yaml
matchLabels:
  tier: frontend
```

In the ReplicaSet, `.spec.template.metadata.labels` must match `spec.selector`, or it will be rejected by the API.

**Note:** For 2 ReplicaSets specifying the same `.spec.selector` but different `.spec.template.metadata.labels` and `.spec.template.spec` fields, each ReplicaSet ignores the Pods created by the other ReplicaSet.

### Replicas[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicas)

You can specify how many Pods should run concurrently by setting `.spec.replicas`. The ReplicaSet will create/delete its Pods to match this number.

If you do not specify `.spec.replicas`, then it defaults to 1.

## Working with ReplicaSets[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#working-with-replicasets)

### Deleting a ReplicaSet and its Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deleting-a-replicaset-and-its-pods)

To delete a ReplicaSet and all of its Pods, use [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete). The [Garbage collector](https://kubernetes.io/docs/concepts/architecture/garbage-collection/) automatically deletes all of the dependent Pods by default.

When using the REST API or the `client-go` library, you must set `propagationPolicy` to `Background` or `Foreground` in the `-d` option. For example:

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
  -H "Content-Type: application/json"
```

### Deleting just a ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deleting-just-a-replicaset)

You can delete a ReplicaSet without affecting any of its Pods using [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete) with the `--cascade=orphan` option. When using the REST API or the `client-go` library, you must set `propagationPolicy` to `Orphan`. For example:

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
  -H "Content-Type: application/json"
```

Once the original is deleted, you can create a new ReplicaSet to replace it. As long as the old and new `.spec.selector` are the same, then the new one will adopt the old Pods. However, it will not make any effort to make existing Pods match a new, different pod template. To update Pods to a new spec in a controlled way, use a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment), as ReplicaSets do not support a rolling update directly.

### Isolating Pods from a ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#isolating-pods-from-a-replicaset)

You can remove Pods from a ReplicaSet by changing their labels. This technique may be used to remove Pods from service for debugging, data recovery, etc. Pods that are removed in this way will be replaced automatically ( assuming that the number of replicas is not also changed).

### Scaling a ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#scaling-a-replicaset)

A ReplicaSet can be easily scaled up or down by simply updating the `.spec.replicas` field. The ReplicaSet controller ensures that a desired number of Pods with a matching label selector are available and operational.

When scaling down, the ReplicaSet controller chooses which pods to delete by sorting the available pods to prioritize scaling down pods based on the following general algorithm:

1. Pending (and unschedulable) pods are scaled down first
2. If `controller.kubernetes.io/pod-deletion-cost` annotation is set, then the pod with the lower value will come first.
3. Pods on nodes with more replicas come before pods on nodes with fewer replicas.
4. If the pods' creation times differ, the pod that was created more recently comes before the older pod (the creation times are bucketed on an integer log scale when the `LogarithmicScaleDown` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) is enabled)

If all of the above match, then selection is random.

### Pod deletion cost[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-deletion-cost)

**FEATURE STATE:** `Kubernetes v1.22 [beta]`

Using the [`controller.kubernetes.io/pod-deletion-cost`](https://kubernetes.io/docs/reference/labels-annotations-taints/#pod-deletion-cost) annotation, users can set a preference regarding which pods to remove first when downscaling a ReplicaSet.

The annotation should be set on the pod, the range is [-2147483647, 2147483647]. It represents the cost of deleting a pod compared to other pods belonging to the same ReplicaSet. Pods with lower deletion cost are preferred to be deleted before pods with higher deletion cost.

The implicit value for this annotation for pods that don't set it is 0; negative values are permitted. Invalid values will be rejected by the API server.

This feature is beta and enabled by default. You can disable it using the [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) `PodDeletionCost` in both kube-apiserver and kube-controller-manager.

**Note:**

- This is honored on a best-effort basis, so it does not offer any guarantees on pod deletion order.
- Users should avoid updating the annotation frequently, such as updating it based on a metric value, because doing so will generate a significant number of pod updates on the apiserver.

#### Example Use Case[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#example-use-case)

The different pods of an application could have different utilization levels. On scale down, the application may prefer to remove the pods with lower utilization. To avoid frequently updating the pods, the application should update `controller.kubernetes.io/pod-deletion-cost` once before issuing a scale down (setting the annotation to a value proportional to pod utilization level). This works if the application itself controls the down scaling; for example, the driver pod of a Spark deployment.

### ReplicaSet as a Horizontal Pod Autoscaler Target[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicaset-as-a-horizontal-pod-autoscaler-target)

A ReplicaSet can also be a target for [Horizontal Pod Autoscalers (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). That is, a ReplicaSet can be auto-scaled by an HPA. Here is an example HPA targeting the ReplicaSet we created in the previous example.

[`controllers/hpa-rs.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/hpa-rs.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/hpa-rs.yaml to clipboard")

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

Saving this manifest into `hpa-rs.yaml` and submitting it to a Kubernetes cluster should create the defined HPA that autoscales the target ReplicaSet depending on the CPU usage of the replicated Pods.

```shell
kubectl apply -f https://k8s.io/examples/controllers/hpa-rs.yaml
```

Alternatively, you can use the `kubectl autoscale` command to accomplish the same (and it's easier!)

```shell
kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50
```

## Alternatives to ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#alternatives-to-replicaset)

### Deployment (recommended)[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-recommended)

[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is an object which can own ReplicaSets and update them and their Pods via declarative, server-side rolling updates. While ReplicaSets can be used independently, today they're mainly used by Deployments as a mechanism to orchestrate Pod creation, deletion and updates. When you use Deployments you don't have to worry about managing the ReplicaSets that they create. Deployments own and manage their ReplicaSets. As such, it is recommended to use Deployments when you want ReplicaSets.

### Bare Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#bare-pods)

Unlike the case where a user directly created Pods, a ReplicaSet replaces Pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicaSet even if your application requires only a single Pod. Think of it similarly to a process supervisor, only it supervises multiple Pods across multiple nodes instead of individual processes on a single node. A ReplicaSet delegates local container restarts to some agent on the node such as Kubelet.

### Job[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job)

Use a [`Job`](https://kubernetes.io/docs/concepts/workloads/controllers/job/) instead of a ReplicaSet for Pods that are expected to terminate on their own (that is, batch jobs).

### DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#daemonset)

Use a [`DaemonSet`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicaSet for Pods that provide a machine-level function, such as machine monitoring or machine logging. These Pods have a lifetime that is tied to a machine lifetime: the Pod needs to be running on the machine before other Pods start, and are safe to terminate when the machine is otherwise ready to be rebooted/shutdown.

### ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicationcontroller)

ReplicaSets are the successors to [ReplicationControllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/). The two serve the same purpose, and behave similarly, except that a ReplicationController does not support set-based selector requirements as described in the [labels user guide](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors). As such, ReplicaSets are preferred over ReplicationControllers

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn about [Pods](https://kubernetes.io/docs/concepts/workloads/pods).
- Learn about [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).
- [Run a Stateless Application Using a Deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/), which relies on ReplicaSets to work.
- `ReplicaSet` is a top-level resource in the Kubernetes REST API. Read the [ReplicaSet](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/replica-set-v1/) object definition to understand the API for replica sets.
- Read about [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) and how you can use it to manage application availability during disruptions.

# 3 - StatefulSets[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-6d72299952c37ca8cc61b416e5bdbcd4)

StatefulSet is the workload API object used to manage stateful applications.

Manages the deployment and scaling of a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), _and provides guarantees about the ordering and uniqueness_ of these Pods.

Like a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of its Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution. Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.

## Using StatefulSets[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#using-statefulsets)

StatefulSets are valuable for applications that require one or more of the following.

- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

In the above, stable is synonymous with persistence across Pod (re)scheduling. If an application doesn't require any stable identifiers or ordered deployment, deletion, or scaling, you should deploy your application using a workload object that provides a set of stateless replicas. [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) may be better suited to your stateless needs.

## Limitations[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#limitations)

- The storage for a given Pod must either be provisioned by a [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md) based on the requested `storage class`, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will _not_ delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
- StatefulSets currently require a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods. You are responsible for creating this Service.
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
- When using [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-updates) with the default [Pod Management Policy](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-management-policies) (`OrderedReady`), it's possible to get into a broken state that requires [manual intervention to repair](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#forced-rollback).

## Components[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#components)

The example below demonstrates the components of a StatefulSet.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

In the above example:

- A Headless Service, named `nginx`, is used to control the network domain.
- The StatefulSet, named `web`, has a Spec that indicates that 3 replicas of the nginx container will be launched in unique Pods.
- The `volumeClaimTemplates` will provide stable storage using [PersistentVolumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) provisioned by a PersistentVolume Provisioner.

The name of a StatefulSet object must be a valid [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-label-names).

### Pod Selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)

You must set the `.spec.selector` field of a StatefulSet to match the labels of its `.spec.template.metadata.labels`. Failing to specify a matching Pod Selector will result in a validation error during StatefulSet creation.

### Volume Claim Templates[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#volume-claim-templates)

You can set the `.spec.volumeClaimTemplates` which can provide stable storage using [PersistentVolumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) provisioned by a PersistentVolume Provisioner.

### Minimum ready seconds[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#minimum-ready-seconds)

**FEATURE STATE:** `Kubernetes v1.25 [stable]`

`.spec.minReadySeconds` is an optional field that specifies the minimum number of seconds for which a newly created Pod should be running and ready without any of its containers crashing, for it to be considered available. This is used to check progression of a rollout when using a [Rolling Update](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-updates) strategy. This field defaults to 0 (the Pod will be considered available as soon as it is ready). To learn more about when a Pod is considered ready, see [Container Probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes).

## Pod Identity[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-identity)

StatefulSet Pods have a unique identity that consists of an ordinal, a stable network identity, and stable storage. The identity sticks to the Pod, regardless of which node it's (re)scheduled on.

### Ordinal Index[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#ordinal-index)

For a StatefulSet with N [replicas](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicas), each Pod in the StatefulSet will be assigned an integer ordinal, that is unique over the Set. By default, pods will be assigned ordinals from 0 up through N-1.

### Start ordinal[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#start-ordinal)

**FEATURE STATE:** `Kubernetes v1.27 [beta]`

`.spec.ordinals` is an optional field that allows you to configure the integer ordinals assigned to each Pod. It defaults to nil. You must enable the `StatefulSetStartOrdinal` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) to use this field. Once enabled, you can configure the following options:

- `.spec.ordinals.start`: If the `.spec.ordinals.start` field is set, Pods will be assigned ordinals from `.spec.ordinals.start` up through `.spec.ordinals.start + .spec.replicas - 1`.

### Stable Network ID[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#stable-network-id)

Each Pod in a StatefulSet derives its hostname from the name of the StatefulSet and the ordinal of the Pod. The pattern for the constructed hostname is `$(statefulset name)-$(ordinal)`. The example above will create three Pods named `web-0,web-1,web-2`. A StatefulSet can use a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to control the domain of its Pods. The domain managed by this Service takes the form: `$(service name).$(namespace).svc.cluster.local`, where "cluster.local" is the cluster domain. As each Pod is created, it gets a matching DNS subdomain, taking the form: `$(podname).$(governing service domain)`, where the governing service is defined by the `serviceName` field on the StatefulSet.

Depending on how DNS is configured in your cluster, you may not be able to look up the DNS name for a newly-run Pod immediately. This behavior can occur when other clients in the cluster have already sent queries for the hostname of the Pod before it was created. Negative caching (normal in DNS) means that the results of previous failed lookups are remembered and reused, even after the Pod is running, for at least a few seconds.

If you need to discover Pods promptly after they are created, you have a few options:

- Query the Kubernetes API directly (for example, using a watch) rather than relying on DNS lookups.
- Decrease the time of caching in your Kubernetes DNS provider (typically this means editing the config map for CoreDNS, which currently caches for 30 seconds).

As mentioned in the [limitations](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#limitations) section, you are responsible for creating the [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) responsible for the network identity of the pods.

Here are some examples of choices for Cluster Domain, Service name, StatefulSet name, and how that affects the DNS names for the StatefulSet's Pods.

|Cluster Domain|Service (ns/name)|StatefulSet (ns/name)|StatefulSet Domain|Pod DNS|Pod Hostname|
|---|---|---|---|---|---|
|cluster.local|default/nginx|default/web|nginx.default.svc.cluster.local|web-{0..N-1}.nginx.default.svc.cluster.local|web-{0..N-1}|
|cluster.local|foo/nginx|foo/web|nginx.foo.svc.cluster.local|web-{0..N-1}.nginx.foo.svc.cluster.local|web-{0..N-1}|
|kube.local|foo/nginx|foo/web|nginx.foo.svc.kube.local|web-{0..N-1}.nginx.foo.svc.kube.local|web-{0..N-1}|

**Note:** Cluster Domain will be set to `cluster.local` unless [otherwise configured](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

### Stable Storage[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#stable-storage)

For each VolumeClaimTemplate entry defined in a StatefulSet, each Pod receives one PersistentVolumeClaim. In the nginx example above, each Pod receives a single PersistentVolume with a StorageClass of `my-storage-class` and 1 Gib of provisioned storage. If no StorageClass is specified, then the default StorageClass will be used. When a Pod is (re)scheduled onto a node, its `volumeMounts` mount the PersistentVolumes associated with its PersistentVolume Claims. Note that, the PersistentVolumes associated with the Pods' PersistentVolume Claims are not deleted when the Pods, or StatefulSet are deleted. This must be done manually.

### Pod Name Label[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-name-label)

When the StatefulSet [controller](https://kubernetes.io/docs/concepts/architecture/controller/) creates a Pod, it adds a label, `statefulset.kubernetes.io/pod-name`, that is set to the name of the Pod. This label allows you to attach a Service to a specific Pod in the StatefulSet.

## Deployment and Scaling Guarantees[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-and-scaling-guarantees)

- For a StatefulSet with N replicas, when Pods are being deployed, they are created sequentially, in order from {0..N-1}.
- When Pods are being deleted, they are terminated in reverse order, from {N-1..0}.
- Before a scaling operation is applied to a Pod, all of its predecessors must be Running and Ready.
- Before a Pod is terminated, all of its successors must be completely shutdown.

The StatefulSet should not specify a `pod.Spec.TerminationGracePeriodSeconds` of 0. This practice is unsafe and strongly discouraged. For further explanation, please refer to [force deleting StatefulSet Pods](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/).

When the nginx example above is created, three Pods will be deployed in the order web-0, web-1, web-2. web-1 will not be deployed before web-0 is [Running and Ready](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/), and web-2 will not be deployed until web-1 is Running and Ready. If web-0 should fail, after web-1 is Running and Ready, but before web-2 is launched, web-2 will not be launched until web-0 is successfully relaunched and becomes Running and Ready.

If a user were to scale the deployed example by patching the StatefulSet such that `replicas=1`, web-2 would be terminated first. web-1 would not be terminated until web-2 is fully shutdown and deleted. If web-0 were to fail after web-2 has been terminated and is completely shutdown, but prior to web-1's termination, web-1 would not be terminated until web-0 is Running and Ready.

### Pod Management Policies[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-management-policies)

StatefulSet allows you to relax its ordering guarantees while preserving its uniqueness and identity guarantees via its `.spec.podManagementPolicy` field.

#### OrderedReady Pod Management[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#orderedready-pod-management)

`OrderedReady` pod management is the default for StatefulSets. It implements the behavior described [above](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-and-scaling-guarantees).

#### Parallel Pod Management[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#parallel-pod-management)

`Parallel` pod management tells the StatefulSet controller to launch or terminate all Pods in parallel, and to not wait for Pods to become Running and Ready or completely terminated prior to launching or terminating another Pod. This option only affects the behavior for scaling operations. Updates are not affected.

## Update strategies[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#update-strategies)

A StatefulSet's `.spec.updateStrategy` field allows you to configure and disable automated rolling updates for containers, labels, resource request/limits, and annotations for the Pods in a StatefulSet. There are two possible values:

`OnDelete`

When a StatefulSet's `.spec.updateStrategy.type` is set to `OnDelete`, the StatefulSet controller will not automatically update the Pods in a StatefulSet. Users must manually delete Pods to cause the controller to create new Pods that reflect modifications made to a StatefulSet's `.spec.template`.

`RollingUpdate`

The `RollingUpdate` update strategy implements automated, rolling updates for the Pods in a StatefulSet. This is the default update strategy.

## Rolling Updates[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-updates)

When a StatefulSet's `.spec.updateStrategy.type` is set to `RollingUpdate`, the StatefulSet controller will delete and recreate each Pod in the StatefulSet. It will proceed in the same order as Pod termination (from the largest ordinal to the smallest), updating each Pod one at a time.

The Kubernetes control plane waits until an updated Pod is Running and Ready prior to updating its predecessor. If you have set `.spec.minReadySeconds` (see [Minimum Ready Seconds](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#minimum-ready-seconds)), the control plane additionally waits that amount of time after the Pod turns ready, before moving on.

### Partitioned rolling updates[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#partitions)

The `RollingUpdate` update strategy can be partitioned, by specifying a `.spec.updateStrategy.rollingUpdate.partition`. If a partition is specified, all Pods with an ordinal that is greater than or equal to the partition will be updated when the StatefulSet's `.spec.template` is updated. All Pods with an ordinal that is less than the partition will not be updated, and, even if they are deleted, they will be recreated at the previous version. If a StatefulSet's `.spec.updateStrategy.rollingUpdate.partition` is greater than its `.spec.replicas`, updates to its `.spec.template` will not be propagated to its Pods. In most cases you will not need to use a partition, but they are useful if you want to stage an update, roll out a canary, or perform a phased roll out.

### Maximum unavailable Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#maximum-unavailable-pods)

**FEATURE STATE:** `Kubernetes v1.24 [alpha]`

You can control the maximum number of Pods that can be unavailable during an update by specifying the `.spec.updateStrategy.rollingUpdate.maxUnavailable` field. The value can be an absolute number (for example, `5`) or a percentage of desired Pods (for example, `10%`). Absolute number is calculated from the percentage value by rounding it up. This field cannot be 0. The default setting is 1.

This field applies to all Pods in the range `0` to `replicas - 1`. If there is any unavailable Pod in the range `0` to `replicas - 1`, it will be counted towards `maxUnavailable`.

**Note:** The `maxUnavailable` field is in Alpha stage and it is honored only by API servers that are running with the `MaxUnavailableStatefulSet` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) enabled.

### Forced rollback[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#forced-rollback)

When using [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-updates) with the default [Pod Management Policy](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-management-policies) (`OrderedReady`), it's possible to get into a broken state that requires manual intervention to repair.

If you update the Pod template to a configuration that never becomes Running and Ready (for example, due to a bad binary or application-level configuration error), StatefulSet will stop the rollout and wait.

In this state, it's not enough to revert the Pod template to a good configuration. Due to a [known issue](https://github.com/kubernetes/kubernetes/issues/67250), StatefulSet will continue to wait for the broken Pod to become Ready (which never happens) before it will attempt to revert it back to the working configuration.

After reverting the template, you must also delete any Pods that StatefulSet had already attempted to run with the bad configuration. StatefulSet will then begin to recreate the Pods using the reverted template.

## PersistentVolumeClaim retention[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#persistentvolumeclaim-retention)

**FEATURE STATE:** `Kubernetes v1.27 [beta]`

The optional `.spec.persistentVolumeClaimRetentionPolicy` field controls if and how PVCs are deleted during the lifecycle of a StatefulSet. You must enable the `StatefulSetAutoDeletePVC` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) on the API server and the controller manager to use this field. Once enabled, there are two policies you can configure for each StatefulSet:

`whenDeleted`

configures the volume retention behavior that applies when the StatefulSet is deleted

`whenScaled`

configures the volume retention behavior that applies when the replica count of the StatefulSet is reduced; for example, when scaling down the set.

For each policy that you can configure, you can set the value to either `Delete` or `Retain`.

`Delete`

The PVCs created from the StatefulSet `volumeClaimTemplate` are deleted for each Pod affected by the policy. With the `whenDeleted` policy all PVCs from the `volumeClaimTemplate` are deleted after their Pods have been deleted. With the `whenScaled` policy, only PVCs corresponding to Pod replicas being scaled down are deleted, after their Pods have been deleted.

`Retain` (default)

PVCs from the `volumeClaimTemplate` are not affected when their Pod is deleted. This is the behavior before this new feature.

Bear in mind that these policies **only** apply when Pods are being removed due to the StatefulSet being deleted or scaled down. For example, if a Pod associated with a StatefulSet fails due to node failure, and the control plane creates a replacement Pod, the StatefulSet retains the existing PVC. The existing volume is unaffected, and the cluster will attach it to the node where the new Pod is about to launch.

The default for policies is `Retain`, matching the StatefulSet behavior before this new feature.

Here is an example policy.

```yaml
apiVersion: apps/v1
kind: StatefulSet
...
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Delete
...
```

The StatefulSet [controller](https://kubernetes.io/docs/concepts/architecture/controller/) adds [owner references](https://kubernetes.io/docs/concepts/overview/working-with-objects/owners-dependents/#owner-references-in-object-specifications) to its PVCs, which are then deleted by the [garbage collector](https://kubernetes.io/docs/concepts/architecture/garbage-collection/) after the Pod is terminated. This enables the Pod to cleanly unmount all volumes before the PVCs are deleted (and before the backing PV and volume are deleted, depending on the retain policy). When you set the `whenDeleted` policy to `Delete`, an owner reference to the StatefulSet instance is placed on all PVCs associated with that StatefulSet.

The `whenScaled` policy must delete PVCs only when a Pod is scaled down, and not when a Pod is deleted for another reason. When reconciling, the StatefulSet controller compares its desired replica count to the actual Pods present on the cluster. Any StatefulSet Pod whose id greater than the replica count is condemned and marked for deletion. If the `whenScaled` policy is `Delete`, the condemned Pods are first set as owners to the associated StatefulSet template PVCs, before the Pod is deleted. This causes the PVCs to be garbage collected after only the condemned Pods have terminated.

This means that if the controller crashes and restarts, no Pod will be deleted before its owner reference has been updated appropriate to the policy. If a condemned Pod is force-deleted while the controller is down, the owner reference may or may not have been set up, depending on when the controller crashed. It may take several reconcile loops to update the owner references, so some condemned Pods may have set up owner references and others may not. For this reason we recommend waiting for the controller to come back up, which will verify owner references before terminating Pods. If that is not possible, the operator should verify the owner references on PVCs to ensure the expected objects are deleted when Pods are force-deleted.

### Replicas[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicas)

`.spec.replicas` is an optional field that specifies the number of desired Pods. It defaults to 1.

Should you manually scale a deployment, example via `kubectl scale statefulset statefulset --replicas=X`, and then you update that StatefulSet based on a manifest (for example: by running `kubectl apply -f statefulset.yaml`), then applying that manifest overwrites the manual scaling that you previously did.

If a [HorizontalPodAutoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) (or any similar API for horizontal scaling) is managing scaling for a Statefulset, don't set `.spec.replicas`. Instead, allow the Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) to manage the `.spec.replicas` field automatically.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn about [Pods](https://kubernetes.io/docs/concepts/workloads/pods).
- Find out how to use StatefulSets
    - Follow an example of [deploying a stateful application](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/).
    - Follow an example of [deploying Cassandra with Stateful Sets](https://kubernetes.io/docs/tutorials/stateful-application/cassandra/).
    - Follow an example of [running a replicated stateful application](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/).
    - Learn how to [scale a StatefulSet](https://kubernetes.io/docs/tasks/run-application/scale-stateful-set/).
    - Learn what's involved when you [delete a StatefulSet](https://kubernetes.io/docs/tasks/run-application/delete-stateful-set/).
    - Learn how to [configure a Pod to use a volume for storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/).
    - Learn how to [configure a Pod to use a PersistentVolume for storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/).
- `StatefulSet` is a top-level resource in the Kubernetes REST API. Read the [StatefulSet](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/stateful-set-v1/) object definition to understand the API for stateful sets.
- Read about [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) and how you can use it to manage application availability during disruptions.

# 4 - DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-41600eb8b6631c88848156f381e9d588)

A _DaemonSet_ ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a DaemonSet are:

- running a cluster storage daemon on every node
- running a logs collection daemon on every node
- running a node monitoring daemon on every node

In a simple case, one DaemonSet, covering all nodes, would be used for each type of daemon. A more complex setup might use multiple DaemonSets for a single type of daemon, but with different flags and/or different memory and cpu requests for different hardware types.

## Writing a DaemonSet Spec[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-daemonset-spec)

### Create a DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#create-a-daemonset)

You can describe a DaemonSet in a YAML file. For example, the `daemonset.yaml` file below describes a DaemonSet that runs the fluentd-elasticsearch Docker image:

[`controllers/daemonset.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/daemonset.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/daemonset.yaml to clipboard")

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

Create a DaemonSet based on the YAML file:

```
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```

### Required Fields[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#required-fields)

As with all other Kubernetes config, a DaemonSet needs `apiVersion`, `kind`, and `metadata` fields. For general information about working with config files, see [running stateless applications](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/) and [object management using kubectl](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/).

The name of a DaemonSet object must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

A DaemonSet also needs a [`.spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) section.

### Pod Template[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-template)

The `.spec.template` is one of the required fields in `.spec`.

The `.spec.template` is a [pod template](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/), except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a Pod template in a DaemonSet has to specify appropriate labels (see [pod selector](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)).

A Pod Template in a DaemonSet must have a [`RestartPolicy`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Always`, or be unspecified, which defaults to `Always`.

### Pod Selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)

The `.spec.selector` field is a pod selector. It works the same as the `.spec.selector` of a [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/).

You must specify a pod selector that matches the labels of the `.spec.template`. Also, once a DaemonSet is created, its `.spec.selector` can not be mutated. Mutating the pod selector can lead to the unintentional orphaning of Pods, and it was found to be confusing to users.

The `.spec.selector` is an object consisting of two fields:

- `matchLabels` - works the same as the `.spec.selector` of a [ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/).
- `matchExpressions` - allows to build more sophisticated selectors by specifying key, list of values and an operator that relates the key and values.

When the two are specified the result is ANDed.

The `.spec.selector` must match the `.spec.template.metadata.labels`. Config with these two not matching will be rejected by the API.

### Running Pods on select Nodes[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#running-pods-on-select-nodes)

If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that [node selector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/). Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that [node affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/). If you do not specify either, then the DaemonSet controller will create Pods on all nodes.

## How Daemon Pods are scheduled[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#how-daemon-pods-are-scheduled)

A DaemonSet ensures that all eligible nodes run a copy of a Pod. The DaemonSet controller creates a Pod for each eligible node and adds the `spec.affinity.nodeAffinity` field of the Pod to match the target host. After the Pod is created, the default scheduler typically takes over and then binds the Pod to the target host by setting the `.spec.nodeName` field. If the new Pod cannot fit on the node, the default scheduler may preempt (evict) some of the existing Pods based on the [priority](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#pod-priority) of the new Pod.

The user can specify a different scheduler for the Pods of the DaemonSet, by setting the `.spec.template.spec.schedulerName` field of the DaemonSet.

The original node affinity specified at the `.spec.template.spec.affinity.nodeAffinity` field (if specified) is taken into consideration by the DaemonSet controller when evaluating the eligible nodes, but is replaced on the created Pod with the node affinity that matches the name of the eligible node.

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchFields:
      - key: metadata.name
        operator: In
        values:
        - target-host-name
```

### Taints and tolerations[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#taints-and-tolerations)

The DaemonSet controller automatically adds a set of [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) to DaemonSet Pods:

|Toleration key|Effect|Details|
|---|---|---|
|[`node.kubernetes.io/not-ready`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-not-ready)|`NoExecute`|DaemonSet Pods can be scheduled onto nodes that are not healthy or ready to accept Pods. Any DaemonSet Pods running on such nodes will not be evicted.|
|[`node.kubernetes.io/unreachable`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-unreachable)|`NoExecute`|DaemonSet Pods can be scheduled onto nodes that are unreachable from the node controller. Any DaemonSet Pods running on such nodes will not be evicted.|
|[`node.kubernetes.io/disk-pressure`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-disk-pressure)|`NoSchedule`|DaemonSet Pods can be scheduled onto nodes with disk pressure issues.|
|[`node.kubernetes.io/memory-pressure`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-memory-pressure)|`NoSchedule`|DaemonSet Pods can be scheduled onto nodes with memory pressure issues.|
|[`node.kubernetes.io/pid-pressure`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-pid-pressure)|`NoSchedule`|DaemonSet Pods can be scheduled onto nodes with process pressure issues.|
|[`node.kubernetes.io/unschedulable`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-unschedulable)|`NoSchedule`|DaemonSet Pods can be scheduled onto nodes that are unschedulable.|
|[`node.kubernetes.io/network-unavailable`](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-network-unavailable)|`NoSchedule`|**Only added for DaemonSet Pods that request host networking**, i.e., Pods having `spec.hostNetwork: true`. Such DaemonSet Pods can be scheduled onto nodes with unavailable network.|

You can add your own tolerations to the Pods of a DaemonSet as well, by defining these in the Pod template of the DaemonSet.

Because the DaemonSet controller sets the `node.kubernetes.io/unschedulable:NoSchedule` toleration automatically, Kubernetes can run DaemonSet Pods on nodes that are marked as _unschedulable_.

If you use a DaemonSet to provide an important node-level function, such as [cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/), it is helpful that Kubernetes places DaemonSet Pods on nodes before they are ready. For example, without that special toleration, you could end up in a deadlock situation where the node is not marked as ready because the network plugin is not running there, and at the same time the network plugin is not running on that node because the node is not yet ready.

## Communicating with Daemon Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#communicating-with-daemon-pods)

Some possible patterns for communicating with Pods in a DaemonSet are:

- **Push**: Pods in the DaemonSet are configured to send updates to another service, such as a stats database. They do not have clients.
- **NodeIP and Known Port**: Pods in the DaemonSet can use a `hostPort`, so that the pods are reachable via the node IPs. Clients know the list of node IPs somehow, and know the port by convention.
- **DNS**: Create a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) with the same pod selector, and then discover DaemonSets using the `endpoints` resource or retrieve multiple A records from DNS.
- **Service**: Create a service with the same Pod selector, and use the service to reach a daemon on a random node. (No way to reach specific node.)

## Updating a DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#updating-a-daemonset)

If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete Pods from newly not-matching nodes.

You can modify the Pods that a DaemonSet creates. However, Pods do not allow all fields to be updated. Also, the DaemonSet controller will use the original template the next time a node (even with the same name) is created.

You can delete a DaemonSet. If you specify `--cascade=orphan` with `kubectl`, then the Pods will be left on the nodes. If you subsequently create a new DaemonSet with the same selector, the new DaemonSet adopts the existing Pods. If any Pods need replacing the DaemonSet replaces them according to its `updateStrategy`.

You can [perform a rolling update](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/) on a DaemonSet.

## Alternatives to DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#alternatives-to-daemonset)

### Init scripts[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#init-scripts)

It is certainly possible to run daemon processes by directly starting them on a node (e.g. using `init`, `upstartd`, or `systemd`). This is perfectly fine. However, there are several advantages to running such processes via a DaemonSet:

- Ability to monitor and manage logs for daemons in the same way as applications.
- Same config language and tools (e.g. Pod templates, `kubectl`) for daemons and applications.
- Running daemons in containers with resource limits increases isolation between daemons from app containers. However, this can also be accomplished by running the daemons in a container but not in a Pod.

### Bare Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#bare-pods)

It is possible to create Pods directly which specify a particular node to run on. However, a DaemonSet replaces Pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, you should use a DaemonSet rather than creating individual Pods.

### Static Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#static-pods)

It is possible to create Pods by writing a file to a certain directory watched by Kubelet. These are called [static pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/). Unlike DaemonSet, static Pods cannot be managed with kubectl or other Kubernetes API clients. Static Pods do not depend on the apiserver, making them useful in cluster bootstrapping cases. Also, static Pods may be deprecated in the future.

### Deployments[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployments)

DaemonSets are similar to [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) in that they both create Pods, and those Pods have processes which are not expected to terminate (e.g. web servers, storage servers).

Use a Deployment for stateless services, like frontends, where scaling up and down the number of replicas and rolling out updates are more important than controlling exactly which host the Pod runs on. Use a DaemonSet when it is important that a copy of a Pod always run on all or certain hosts, if the DaemonSet provides node-level functionality that allows other Pods to run correctly on that particular node.

For example, [network plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) often include a component that runs as a DaemonSet. The DaemonSet component makes sure that the node where it's running has working cluster networking.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn about [Pods](https://kubernetes.io/docs/concepts/workloads/pods).
    - Learn about [static Pods](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#static-pods), which are useful for running Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) components.
- Find out how to use DaemonSets
    - [Perform a rolling update on a DaemonSet](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)
    - [Perform a rollback on a DaemonSet](https://kubernetes.io/docs/tasks/manage-daemon/rollback-daemon-set/) (for example, if a roll out didn't work how you expected).
- Understand [how Kubernetes assigns Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).
- Learn about [device plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/) and [add ons](https://kubernetes.io/docs/concepts/cluster-administration/addons/), which often run as DaemonSets.
- `DaemonSet` is a top-level resource in the Kubernetes REST API. Read the [DaemonSet](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/daemon-set-v1/) object definition to understand the API for daemon sets.

# 5 - Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-cc7cc3c4907039d9f863162e20bfbbef)

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).

You can also use a Job to run multiple Pods in parallel.

If you want to run a Job (either a single task, or several in parallel) on a schedule, see [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/).

## Running an example Job[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#running-an-example-job)

Here is an example Job config. It computes π to 2000 places and prints it out. It takes around 10s to complete.

[`controllers/job.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/job.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/job.yaml to clipboard")

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

```

You can run the example with this command:

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/job.yaml
```

The output is similar to this:

```
job.batch/pi created
```

Check on the status of the Job with `kubectl`:

- [kubectl describe job pi](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#check-status-of-job-0)
- [kubectl get job pi -o yaml](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#check-status-of-job-1)

```bash

Name:           pi
Namespace:      default
Selector:       batch.kubernetes.io/controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
Labels:         batch.kubernetes.io/controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
                batch.kubernetes.io/job-name=pi
                ...
Annotations:    batch.kubernetes.io/job-tracking: ""
Parallelism:    1
Completions:    1
Start Time:     Mon, 02 Dec 2019 15:20:11 +0200
Completed At:   Mon, 02 Dec 2019 15:21:16 +0200
Duration:       65s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  batch.kubernetes.io/controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
           batch.kubernetes.io/job-name=pi
  Containers:
   pi:
    Image:      perl:5.34.0
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  21s   job-controller  Created pod: pi-xf9p4
  Normal  Completed         18s   job-controller  Job completed
```

To view completed Pods of a Job, use `kubectl get pods`.

To list all the Pods that belong to a Job in a machine readable form, you can use a command like this:

```shell
pods=$(kubectl get pods --selector=batch.kubernetes.io/job-name=pi --output=jsonpath='{.items[*].metadata.name}')
echo $pods
```

The output is similar to this:

```
pi-5rwd7
```

Here, the selector is the same as the selector for the Job. The `--output=jsonpath` option specifies an expression with the name from each Pod in the returned list.

View the standard output of one of the pods:

```shell
kubectl logs $pods
```

Another way to view the logs of a Job:

```shell
kubectl logs jobs/pi
```

The output is similar to this:

```
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```

## Writing a Job spec[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-job-spec)

As with all other Kubernetes config, a Job needs `apiVersion`, `kind`, and `metadata` fields.

When the control plane creates new Pods for a Job, the `.metadata.name` of the Job is part of the basis for naming those Pods. The name of a Job must be a valid [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-label-names). Even when the name is a DNS subdomain, the name must be no longer than 63 characters.

A Job also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

### Job Labels[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-labels)

Job labels will have `batch.kubernetes.io/` prefix for `job-name` and `controller-uid`.

### Pod Template[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-template)

The `.spec.template` is the only required field of the `.spec`.

The `.spec.template` is a [pod template](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/), except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a pod template in a Job must specify appropriate labels (see [pod selector](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)) and an appropriate restart policy.

Only a [`RestartPolicy`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Never` or `OnFailure` is allowed.

### Pod selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)

The `.spec.selector` field is optional. In almost all cases you should not specify it. See section [specifying your own pod selector](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#specifying-your-own-pod-selector).

### Parallel execution for Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#parallel-jobs)

There are three main types of task suitable to run as a Job:

1. Non-parallel Jobs
    - normally, only one Pod is started, unless the Pod fails.
    - the Job is complete as soon as its Pod terminates successfully.
2. Parallel Jobs with a _fixed completion count_:
    - specify a non-zero positive value for `.spec.completions`.
    - the Job represents the overall task, and is complete when there are `.spec.completions` successful Pods.
    - when using `.spec.completionMode="Indexed"`, each Pod gets a different index in the range 0 to `.spec.completions-1`.
3. Parallel Jobs with a _work queue_:
    - do not specify `.spec.completions`, default to `.spec.parallelism`.
    - the Pods must coordinate amongst themselves or an external service to determine what each should work on. For example, a Pod might fetch a batch of up to N items from the work queue.
    - each Pod is independently capable of determining whether or not all its peers are done, and thus that the entire Job is done.
    - when _any_ Pod from the Job terminates with success, no new Pods are created.
    - once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
    - once any Pod has exited with success, no other Pod should still be doing any work for this task or writing any output. They should all be in the process of exiting.

For a _non-parallel_ Job, you can leave both `.spec.completions` and `.spec.parallelism` unset. When both are unset, both are defaulted to 1.

For a _fixed completion count_ Job, you should set `.spec.completions` to the number of completions needed. You can set `.spec.parallelism`, or leave it unset and it will default to 1.

For a _work queue_ Job, you must leave `.spec.completions` unset, and set `.spec.parallelism` to a non-negative integer.

For more information about how to make use of the different types of job, see the [job patterns](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-patterns) section.

#### Controlling parallelism[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#controlling-parallelism)

The requested parallelism (`.spec.parallelism`) can be set to any non-negative value. If it is unspecified, it defaults to 1. If it is specified as 0, then the Job is effectively paused until it is increased.

Actual parallelism (number of pods running at any instant) may be more or less than requested parallelism, for a variety of reasons:

- For _fixed completion count_ Jobs, the actual number of pods running in parallel will not exceed the number of remaining completions. Higher values of `.spec.parallelism` are effectively ignored.
- For _work queue_ Jobs, no new Pods are started after any Pod has succeeded -- remaining Pods are allowed to complete, however.
- If the Job [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) has not had time to react.
- If the Job controller failed to create Pods for any reason (lack of `ResourceQuota`, lack of permission, etc.), then there may be fewer pods than requested.
- The Job controller may throttle new Pod creation due to excessive previous pod failures in the same Job.
- When a Pod is gracefully shut down, it takes time to stop.

### Completion mode[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#completion-mode)

**FEATURE STATE:** `Kubernetes v1.24 [stable]`

Jobs with _fixed completion count_ - that is, jobs that have non null `.spec.completions` - can have a completion mode that is specified in `.spec.completionMode`:

- `NonIndexed` (default): the Job is considered complete when there have been `.spec.completions` successfully completed Pods. In other words, each Pod completion is homologous to each other. Note that Jobs that have null `.spec.completions` are implicitly `NonIndexed`.
    
- `Indexed`: the Pods of a Job get an associated completion index from 0 to `.spec.completions-1`. The index is available through three mechanisms:
    
    - The Pod annotation `batch.kubernetes.io/job-completion-index`.
    - As part of the Pod hostname, following the pattern `$(job-name)-$(index)`. When you use an Indexed Job in combination with a [Service](https://kubernetes.io/docs/concepts/services-networking/service/), Pods within the Job can use the deterministic hostnames to address each other via DNS. For more information about how to configure this, see [Job with Pod-to-Pod Communication](https://kubernetes.io/docs/tasks/job/job-with-pod-to-pod-communication/).
    - From the containerized task, in the environment variable `JOB_COMPLETION_INDEX`.
    
    The Job is considered complete when there is one successfully completed Pod for each index. For more information about how to use this mode, see [Indexed Job for Parallel Processing with Static Work Assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/).
    

**Note:** Although rare, more than one Pod could be started for the same index (due to various reasons such as node failures, kubelet restarts, or Pod evictions). In this case, only the first Pod that completes successfully will count towards the completion count and update the status of the Job. The other Pods that are running or completed for the same index will be deleted by the Job controller once they are detected.

## Handling Pod and container failures[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#handling-pod-and-container-failures)

A container in a Pod may fail for a number of reasons, such as because the process in it exited with a non-zero exit code, or the container was killed for exceeding a memory limit, etc. If this happens, and the `.spec.template.spec.restartPolicy = "OnFailure"`, then the Pod stays on the node, but the container is re-run. Therefore, your program needs to handle the case when it is restarted locally, or else specify `.spec.template.spec.restartPolicy = "Never"`. See [pod lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#example-states) for more information on `restartPolicy`.

An entire Pod can also fail, for a number of reasons, such as when the pod is kicked off the node (node is upgraded, rebooted, deleted, etc.), or if a container of the Pod fails and the `.spec.template.spec.restartPolicy = "Never"`. When a Pod fails, then the Job controller starts a new Pod. This means that your application needs to handle the case when it is restarted in a new pod. In particular, it needs to handle temporary files, locks, incomplete output and the like caused by previous runs.

By default, each pod failure is counted towards the `.spec.backoffLimit` limit, see [pod backoff failure policy](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-backoff-failure-policy). However, you can customize handling of pod failures by setting the Job's [pod failure policy](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-failure-policy).

Note that even if you specify `.spec.parallelism = 1` and `.spec.completions = 1` and `.spec.template.spec.restartPolicy = "Never"`, the same program may sometimes be started twice.

If you do specify `.spec.parallelism` and `.spec.completions` both greater than 1, then there may be multiple pods running at once. Therefore, your pods must also be tolerant of concurrency.

When the [feature gates](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) `PodDisruptionConditions` and `JobPodFailurePolicy` are both enabled, and the `.spec.podFailurePolicy` field is set, the Job controller does not consider a terminating Pod (a pod that has a `.metadata.deletionTimestamp` field set) as a failure until that Pod is terminal (its `.status.phase` is `Failed` or `Succeeded`). However, the Job controller creates a replacement Pod as soon as the termination becomes apparent. Once the pod terminates, the Job controller evaluates `.backoffLimit` and `.podFailurePolicy` for the relevant Job, taking this now-terminated Pod into consideration.

If either of these requirements is not satisfied, the Job controller counts a terminating Pod as an immediate failure, even if that Pod later terminates with `phase: "Succeeded"`.

### Pod backoff failure policy[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-backoff-failure-policy)

There are situations where you want to fail a Job after some amount of retries due to a logical error in configuration etc. To do so, set `.spec.backoffLimit` to specify the number of retries before considering a Job as failed. The back-off limit is set by default to 6. Failed Pods associated with the Job are recreated by the Job controller with an exponential back-off delay (10s, 20s, 40s ...) capped at six minutes.

The number of retries is calculated in two ways:

- The number of Pods with `.status.phase = "Failed"`.
- When using `restartPolicy = "OnFailure"`, the number of retries in all the containers of Pods with `.status.phase` equal to `Pending` or `Running`.

If either of the calculations reaches the `.spec.backoffLimit`, the Job is considered failed.

**Note:** If your job has `restartPolicy = "OnFailure"`, keep in mind that your Pod running the Job will be terminated once the job backoff limit has been reached. This can make debugging the Job's executable more difficult. We suggest setting `restartPolicy = "Never"` when debugging the Job or using a logging system to ensure output from failed Jobs is not lost inadvertently.

### Pod failure policy[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-failure-policy)

**FEATURE STATE:** `Kubernetes v1.26 [beta]`

**Note:** You can only configure a Pod failure policy for a Job if you have the `JobPodFailurePolicy` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) enabled in your cluster. Additionally, it is recommended to enable the `PodDisruptionConditions` feature gate in order to be able to detect and handle Pod disruption conditions in the Pod failure policy (see also: [Pod disruption conditions](https://kubernetes.io/docs/concepts/workloads/pods/disruptions#pod-disruption-conditions)). Both feature gates are available in Kubernetes 1.27.

A Pod failure policy, defined with the `.spec.podFailurePolicy` field, enables your cluster to handle Pod failures based on the container exit codes and the Pod conditions.

In some situations, you may want to have a better control when handling Pod failures than the control provided by the [Pod backoff failure policy](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-backoff-failure-policy), which is based on the Job's `.spec.backoffLimit`. These are some examples of use cases:

- To optimize costs of running workloads by avoiding unnecessary Pod restarts, you can terminate a Job as soon as one of its Pods fails with an exit code indicating a software bug.
- To guarantee that your Job finishes even if there are disruptions, you can ignore Pod failures caused by disruptions (such [preemption](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#preemption), [API-initiated eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/api-eviction/) or [taint](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)-based eviction) so that they don't count towards the `.spec.backoffLimit` limit of retries.

You can configure a Pod failure policy, in the `.spec.podFailurePolicy` field, to meet the above use cases. This policy can handle Pod failures based on the container exit codes and the Pod conditions.

Here is a manifest for a Job that defines a `podFailurePolicy`:

[`/controllers/job-pod-failure-policy-example.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples//controllers/job-pod-failure-policy-example.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy /controllers/job-pod-failure-policy-example.yaml to clipboard")

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-pod-failure-policy-example
spec:
  completions: 12
  parallelism: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: main
        image: docker.io/library/bash:5
        command: ["bash"]        # example command simulating a bug which triggers the FailJob action
        args:
        - -c
        - echo "Hello world!" && sleep 5 && exit 42
  backoffLimit: 6
  podFailurePolicy:
    rules:
    - action: FailJob
      onExitCodes:
        containerName: main      # optional
        operator: In             # one of: In, NotIn
        values: [42]
    - action: Ignore             # one of: Ignore, FailJob, Count
      onPodConditions:
      - type: DisruptionTarget   # indicates Pod disruption
```

In the example above, the first rule of the Pod failure policy specifies that the Job should be marked failed if the `main` container fails with the 42 exit code. The following are the rules for the `main` container specifically:

- an exit code of 0 means that the container succeeded
- an exit code of 42 means that the **entire Job** failed
- any other exit code represents that the container failed, and hence the entire Pod. The Pod will be re-created if the total number of restarts is below `backoffLimit`. If the `backoffLimit` is reached the **entire Job** failed.

**Note:** Because the Pod template specifies a `restartPolicy: Never`, the kubelet does not restart the `main` container in that particular Pod.

The second rule of the Pod failure policy, specifying the `Ignore` action for failed Pods with condition `DisruptionTarget` excludes Pod disruptions from being counted towards the `.spec.backoffLimit` limit of retries.

**Note:** If the Job failed, either by the Pod failure policy or Pod backoff failure policy, and the Job is running multiple Pods, Kubernetes terminates all the Pods in that Job that are still Pending or Running.

These are some requirements and semantics of the API:

- if you want to use a `.spec.podFailurePolicy` field for a Job, you must also define that Job's pod template with `.spec.restartPolicy` set to `Never`.
- the Pod failure policy rules you specify under `spec.podFailurePolicy.rules` are evaluated in order. Once a rule matches a Pod failure, the remaining rules are ignored. When no rule matches the Pod failure, the default handling applies.
- you may want to restrict a rule to a specific container by specifying its name in`spec.podFailurePolicy.rules[*].containerName`. When not specified the rule applies to all containers. When specified, it should match one the container or `initContainer` names in the Pod template.
- you may specify the action taken when a Pod failure policy is matched by `spec.podFailurePolicy.rules[*].action`. Possible values are:
    - `FailJob`: use to indicate that the Pod's job should be marked as Failed and all running Pods should be terminated.
    - `Ignore`: use to indicate that the counter towards the `.spec.backoffLimit` should not be incremented and a replacement Pod should be created.
    - `Count`: use to indicate that the Pod should be handled in the default way. The counter towards the `.spec.backoffLimit` should be incremented.

**Note:** When you use a `podFailurePolicy`, the job controller only matches Pods in the `Failed` phase. Pods with a deletion timestamp that are not in a terminal phase (`Failed` or `Succeeded`) are considered still terminating. This implies that terminating pods retain a [tracking finalizer](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-tracking-with-finalizers) until they reach a terminal phase. Since Kubernetes 1.27, Kubelet transitions deleted pods to a terminal phase (see: [Pod Phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)). This ensures that deleted pods have their finalizers removed by the Job controller.

## Job termination and cleanup[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-termination-and-cleanup)

When a Job completes, no more Pods are created, but the Pods are [usually](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-backoff-failure-policy) not deleted either. Keeping them around allows you to still view the logs of completed pods to check for errors, warnings, or other diagnostic output. The job object also remains after it is completed so that you can view its status. It is up to the user to delete old jobs after noting their status. Delete the job with `kubectl` (e.g. `kubectl delete jobs/pi` or `kubectl delete -f ./job.yaml`). When you delete the job using `kubectl`, all the pods it created are deleted too.

By default, a Job will run uninterrupted unless a Pod fails (`restartPolicy=Never`) or a Container exits in error (`restartPolicy=OnFailure`), at which point the Job defers to the `.spec.backoffLimit` described above. Once `.spec.backoffLimit` has been reached the Job will be marked as failed and any running Pods will be terminated.

Another way to terminate a Job is by setting an active deadline. Do this by setting the `.spec.activeDeadlineSeconds` field of the Job to a number of seconds. The `activeDeadlineSeconds` applies to the duration of the job, no matter how many Pods are created. Once a Job reaches `activeDeadlineSeconds`, all of its running Pods are terminated and the Job status will become `type: Failed` with `reason: DeadlineExceeded`.

Note that a Job's `.spec.activeDeadlineSeconds` takes precedence over its `.spec.backoffLimit`. Therefore, a Job that is retrying one or more failed Pods will not deploy additional Pods once it reaches the time limit specified by `activeDeadlineSeconds`, even if the `backoffLimit` is not yet reached.

Example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

Note that both the Job spec and the [Pod template spec](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#detailed-behavior) within the Job have an `activeDeadlineSeconds` field. Ensure that you set this field at the proper level.

Keep in mind that the `restartPolicy` applies to the Pod, and not to the Job itself: there is no automatic Job restart once the Job status is `type: Failed`. That is, the Job termination mechanisms activated with `.spec.activeDeadlineSeconds` and `.spec.backoffLimit` result in a permanent Job failure that requires manual intervention to resolve.

## Clean up finished jobs automatically[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#clean-up-finished-jobs-automatically)

Finished Jobs are usually no longer needed in the system. Keeping them around in the system will put pressure on the API server. If the Jobs are managed directly by a higher level controller, such as [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/), the Jobs can be cleaned up by CronJobs based on the specified capacity-based cleanup policy.

### TTL mechanism for finished Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#ttl-mechanism-for-finished-jobs)

**FEATURE STATE:** `Kubernetes v1.23 [stable]`

Another way to clean up finished Jobs (either `Complete` or `Failed`) automatically is to use a TTL mechanism provided by a [TTL controller](https://kubernetes.io/docs/concepts/workloads/controllers/ttlafterfinished/) for finished resources, by specifying the `.spec.ttlSecondsAfterFinished` field of the Job.

When the TTL controller cleans up the Job, it will delete the Job cascadingly, i.e. delete its dependent objects, such as Pods, together with the Job. Note that when the Job is deleted, its lifecycle guarantees, such as finalizers, will be honored.

For example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

The Job `pi-with-ttl` will be eligible to be automatically deleted, `100` seconds after it finishes.

If the field is set to `0`, the Job will be eligible to be automatically deleted immediately after it finishes. If the field is unset, this Job won't be cleaned up by the TTL controller after it finishes.

**Note:**

It is recommended to set `ttlSecondsAfterFinished` field because unmanaged jobs (Jobs that you created directly, and not indirectly through other workload APIs such as CronJob) have a default deletion policy of `orphanDependents` causing Pods created by an unmanaged Job to be left around after that Job is fully deleted. Even though the [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) eventually [garbage collects](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection) the Pods from a deleted Job after they either fail or complete, sometimes those lingering pods may cause cluster performance degradation or in worst case cause the cluster to go offline due to this degradation.

You can use [LimitRanges](https://kubernetes.io/docs/concepts/policy/limit-range/) and [ResourceQuotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) to place a cap on the amount of resources that a particular namespace can consume.

## Job patterns[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-patterns)

The Job object can be used to support reliable parallel execution of Pods. The Job object is not designed to support closely-communicating parallel processes, as commonly found in scientific computing. It does support parallel processing of a set of independent but related _work items_. These might be emails to be sent, frames to be rendered, files to be transcoded, ranges of keys in a NoSQL database to scan, and so on.

In a complex system, there may be multiple different sets of work items. Here we are just considering one set of work items that the user wants to manage together — a _batch job_.

There are several different patterns for parallel computation, each with strengths and weaknesses. The tradeoffs are:

- One Job object for each work item, vs. a single Job object for all work items. The latter is better for large numbers of work items. The former creates some overhead for the user and for the system to manage large numbers of Job objects.
- Number of pods created equals number of work items, vs. each Pod can process multiple work items. The former typically requires less modification to existing code and containers. The latter is better for large numbers of work items, for similar reasons to the previous bullet.
- Several approaches use a work queue. This requires running a queue service, and modifications to the existing program or container to make it use the work queue. Other approaches are easier to adapt to an existing containerised application.

The tradeoffs are summarized here, with columns 2 to 4 corresponding to the above tradeoffs. The pattern names are also links to examples and more detailed description.

|Pattern|Single Job object|Fewer pods than work items?|Use app unmodified?|
|---|---|---|---|
|[Queue with Pod Per Work Item](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)|✓||sometimes|
|[Queue with Variable Pod Count](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)|✓|✓||
|[Indexed Job with Static Work Assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/)|✓||✓|
|[Job Template Expansion](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)|||✓|
|[Job with Pod-to-Pod Communication](https://kubernetes.io/docs/tasks/job/job-with-pod-to-pod-communication/)|✓|sometimes|sometimes|

When you specify completions with `.spec.completions`, each Pod created by the Job controller has an identical [`spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status). This means that all pods for a task will have the same command line and the same image, the same volumes, and (almost) the same environment variables. These patterns are different ways to arrange for pods to work on different things.

This table shows the required settings for `.spec.parallelism` and `.spec.completions` for each of the patterns. Here, `W` is the number of work items.

|Pattern|`.spec.completions`|`.spec.parallelism`|
|---|---|---|
|[Queue with Pod Per Work Item](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)|W|any|
|[Queue with Variable Pod Count](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)|null|any|
|[Indexed Job with Static Work Assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/)|W|any|
|[Job Template Expansion](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)|1|should be 1|
|[Job with Pod-to-Pod Communication](https://kubernetes.io/docs/tasks/job/job-with-pod-to-pod-communication/)|W|W|

## Advanced usage[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#advanced-usage)

### Suspending a Job[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#suspending-a-job)

**FEATURE STATE:** `Kubernetes v1.24 [stable]`

When a Job is created, the Job controller will immediately begin creating Pods to satisfy the Job's requirements and will continue to do so until the Job is complete. However, you may want to temporarily suspend a Job's execution and resume it later, or start Jobs in suspended state and have a custom controller decide later when to start them.

To suspend a Job, you can update the `.spec.suspend` field of the Job to true; later, when you want to resume it again, update it to false. Creating a Job with `.spec.suspend` set to true will create it in the suspended state.

When a Job is resumed from suspension, its `.status.startTime` field will be reset to the current time. This means that the `.spec.activeDeadlineSeconds` timer will be stopped and reset when a Job is suspended and resumed.

When you suspend a Job, any running Pods that don't have a status of `Completed` will be [terminated](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination). with a SIGTERM signal. The Pod's graceful termination period will be honored and your Pod must handle this signal in this period. This may involve saving progress for later or undoing changes. Pods terminated this way will not count towards the Job's `completions` count.

An example Job definition in the suspended state can be like so:

```shell
kubectl get job myjob -o yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  suspend: true
  parallelism: 1
  completions: 5
  template:
    spec:
      ...
```

You can also toggle Job suspension by patching the Job using the command line.

Suspend an active Job:

```shell
kubectl patch job/myjob --type=strategic --patch '{"spec":{"suspend":true}}'
```

Resume a suspended Job:

```shell
kubectl patch job/myjob --type=strategic --patch '{"spec":{"suspend":false}}'
```

The Job's status can be used to determine if a Job is suspended or has been suspended in the past:

```shell
kubectl get jobs/myjob -o yaml
```

```yaml
apiVersion: batch/v1
kind: Job
# .metadata and .spec omitted
status:
  conditions:
  - lastProbeTime: "2021-02-05T13:14:33Z"
    lastTransitionTime: "2021-02-05T13:14:33Z"
    status: "True"
    type: Suspended
  startTime: "2021-02-05T13:13:48Z"
```

The Job condition of type "Suspended" with status "True" means the Job is suspended; the `lastTransitionTime` field can be used to determine how long the Job has been suspended for. If the status of that condition is "False", then the Job was previously suspended and is now running. If such a condition does not exist in the Job's status, the Job has never been stopped.

Events are also created when the Job is suspended and resumed:

```shell
kubectl describe jobs/myjob
```

```
Name:           myjob
...
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  12m   job-controller  Created pod: myjob-hlrpl
  Normal  SuccessfulDelete  11m   job-controller  Deleted pod: myjob-hlrpl
  Normal  Suspended         11m   job-controller  Job suspended
  Normal  SuccessfulCreate  3s    job-controller  Created pod: myjob-jvb44
  Normal  Resumed           3s    job-controller  Job resumed
```

The last four events, particularly the "Suspended" and "Resumed" events, are directly a result of toggling the `.spec.suspend` field. In the time between these two events, we see that no Pods were created, but Pod creation restarted as soon as the Job was resumed.

### Mutable Scheduling Directives[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#mutable-scheduling-directives)

**FEATURE STATE:** `Kubernetes v1.27 [stable]`

In most cases a parallel job will want the pods to run with constraints, like all in the same zone, or all either on GPU model x or y but not a mix of both.

The [suspend](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#suspending-a-job) field is the first step towards achieving those semantics. Suspend allows a custom queue controller to decide when a job should start; However, once a job is unsuspended, a custom queue controller has no influence on where the pods of a job will actually land.

This feature allows updating a Job's scheduling directives before it starts, which gives custom queue controllers the ability to influence pod placement while at the same time offloading actual pod-to-node assignment to kube-scheduler. This is allowed only for suspended Jobs that have never been unsuspended before.

The fields in a Job's pod template that can be updated are node affinity, node selector, tolerations, labels, annotations and [scheduling gates](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/).

### Specifying your own Pod selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#specifying-your-own-pod-selector)

Normally, when you create a Job object, you do not specify `.spec.selector`. The system defaulting logic adds this field when the Job is created. It picks a selector value that will not overlap with any other jobs.

However, in some cases, you might need to override this automatically set selector. To do this, you can specify the `.spec.selector` of the Job.

Be very careful when doing this. If you specify a label selector which is not unique to the pods of that Job, and which matches unrelated Pods, then pods of the unrelated job may be deleted, or this Job may count other Pods as completing it, or one or both Jobs may refuse to create Pods or run to completion. If a non-unique selector is chosen, then other controllers (e.g. ReplicationController) and their Pods may behave in unpredictable ways too. Kubernetes will not stop you from making a mistake when specifying `.spec.selector`.

Here is an example of a case when you might want to use this feature.

Say Job `old` is already running. You want existing Pods to keep running, but you want the rest of the Pods it creates to use a different pod template and for the Job to have a new name. You cannot update the Job because these fields are not updatable. Therefore, you delete Job `old` but _leave its pods running_, using `kubectl delete jobs/old --cascade=orphan`. Before deleting it, you make a note of what selector it uses:

```shell
kubectl get job old -o yaml
```

The output is similar to this:

```yaml
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      batch.kubernetes.io/controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

Then you create a new Job with name `new` and you explicitly specify the same selector. Since the existing Pods have label `batch.kubernetes.io/controller-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`, they are controlled by Job `new` as well.

You need to specify `manualSelector: true` in the new Job since you are not using the selector that the system normally generates for you automatically.

```yaml
kind: Job
metadata:
  name: new
  ...
spec:
  manualSelector: true
  selector:
    matchLabels:
      batch.kubernetes.io/controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

The new Job itself will have a different uid from `a8f3d00d-c6d2-11e5-9f87-42010af00002`. Setting `manualSelector: true` tells the system that you know what you are doing and to allow this mismatch.

### Job tracking with finalizers[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-tracking-with-finalizers)

**FEATURE STATE:** `Kubernetes v1.26 [stable]`

**Note:** The control plane doesn't track Jobs using finalizers, if the Jobs were created when the feature gate `JobTrackingWithFinalizers` was disabled, even after you upgrade the control plane to 1.26.

The control plane keeps track of the Pods that belong to any Job and notices if any such Pod is removed from the API server. To do that, the Job controller creates Pods with the finalizer `batch.kubernetes.io/job-tracking`. The controller removes the finalizer only after the Pod has been accounted for in the Job status, allowing the Pod to be removed by other controllers or users.

Jobs created before upgrading to Kubernetes 1.26 or before the feature gate `JobTrackingWithFinalizers` is enabled are tracked without the use of Pod finalizers. The Job [controller](https://kubernetes.io/docs/concepts/architecture/controller/) updates the status counters for `succeeded` and `failed` Pods based only on the Pods that exist in the cluster. The contol plane can lose track of the progress of the Job if Pods are deleted from the cluster.

You can determine if the control plane is tracking a Job using Pod finalizers by checking if the Job has the annotation `batch.kubernetes.io/job-tracking`. You should **not** manually add or remove this annotation from Jobs. Instead, you can recreate the Jobs to ensure they are tracked using Pod finalizers.

### Elastic Indexed Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#elastic-indexed-jobs)

**FEATURE STATE:** `Kubernetes v1.27 [beta]`

You can scale Indexed Jobs up or down by mutating both `.spec.parallelism` and `.spec.completions` together such that `.spec.parallelism == .spec.completions`. When the `ElasticIndexedJob`[feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) on the [API server](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) is disabled, `.spec.completions` is immutable.

Use cases for elastic Indexed Jobs include batch workloads which require scaling an indexed Job, such as MPI, Horovord, Ray, and PyTorch training jobs.

## Alternatives[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#alternatives)

### Bare Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#bare-pods)

When the node that a Pod is running on reboots or fails, the pod is terminated and will not be restarted. However, a Job will create new Pods to replace terminated ones. For this reason, we recommend that you use a Job rather than a bare Pod, even if your application requires only a single Pod.

### Replication Controller[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replication-controller)

Jobs are complementary to [Replication Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/). A Replication Controller manages Pods which are not expected to terminate (e.g. web servers), and a Job manages Pods that are expected to terminate (e.g. batch tasks).

As discussed in [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/), `Job` is _only_ appropriate for pods with `RestartPolicy` equal to `OnFailure` or `Never`. (Note: If `RestartPolicy` is not set, the default value is `Always`.)

### Single Job starts controller Pod[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#single-job-starts-controller-pod)

Another pattern is for a single Job to create a Pod which then creates other Pods, acting as a sort of custom controller for those Pods. This allows the most flexibility, but may be somewhat complicated to get started with and offers less integration with Kubernetes.

One example of this pattern would be a Job which starts a Pod which runs a script that in turn starts a Spark master controller (see [spark example](https://github.com/kubernetes/examples/tree/master/staging/spark/README.md)), runs a spark driver, and then cleans up.

An advantage of this approach is that the overall process gets the completion guarantee of a Job object, but maintains complete control over what Pods are created and how work is assigned to them.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn about [Pods](https://kubernetes.io/docs/concepts/workloads/pods).
- Read about different ways of running Jobs:
    - [Coarse Parallel Processing Using a Work Queue](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)
    - [Fine Parallel Processing Using a Work Queue](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)
    - Use an [indexed Job for parallel processing with static work assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/)
    - Create multiple Jobs based on a template: [Parallel Processing using Expansions](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)
- Follow the links within [Clean up finished jobs automatically](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#clean-up-finished-jobs-automatically) to learn more about how your cluster can clean up completed and / or failed tasks.
- `Job` is part of the Kubernetes REST API. Read the [Job](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/job-v1/) object definition to understand the API for jobs.
- Read about [`CronJob`](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/), which you can use to define a series of Jobs that will run based on a schedule, similar to the UNIX tool `cron`.
- Practice how to configure handling of retriable and non-retriable pod failures using `podFailurePolicy`, based on the step-by-step [examples](https://kubernetes.io/docs/tasks/job/pod-failure-policy/).

# 6 - Automatic Cleanup for Finished Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-4de50a37ebb6f2340484192126cb7a04)

A time-to-live mechanism to clean up old Jobs that have finished execution.

**FEATURE STATE:** `Kubernetes v1.23 [stable]`

When your Job has finished, it's useful to keep that Job in the API (and not immediately delete the Job) so that you can tell whether the Job succeeded or failed.

Kubernetes' TTL-after-finished [controller](https://kubernetes.io/docs/concepts/architecture/controller/) provides a TTL (time to live) mechanism to limit the lifetime of Job objects that have finished execution.

## Cleanup for finished Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#cleanup-for-finished-jobs)

The TTL-after-finished controller is only supported for Jobs. You can use this mechanism to clean up finished Jobs (either `Complete` or `Failed`) automatically by specifying the `.spec.ttlSecondsAfterFinished` field of a Job, as in this [example](https://kubernetes.io/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically).

The TTL-after-finished controller assumes that a Job is eligible to be cleaned up TTL seconds after the Job has finished. The timer starts once the status condition of the Job changes to show that the Job is either `Complete` or `Failed`; once the TTL has expired, that Job becomes eligible for [cascading](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#cascading-deletion) removal. When the TTL-after-finished controller cleans up a job, it will delete it cascadingly, that is to say it will delete its dependent objects together with it.

Kubernetes honors object lifecycle guarantees on the Job, such as waiting for [finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/).

You can set the TTL seconds at any time. Here are some examples for setting the `.spec.ttlSecondsAfterFinished` field of a Job:

- Specify this field in the Job manifest, so that a Job can be cleaned up automatically some time after it finishes.
- Manually set this field of existing, already finished Jobs, so that they become eligible for cleanup.
- Use a [mutating admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook) to set this field dynamically at Job creation time. Cluster administrators can use this to enforce a TTL policy for finished jobs.
- Use a [mutating admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook) to set this field dynamically after the Job has finished, and choose different TTL values based on job status, labels. For this case, the webhook needs to detect changes to the `.status` of the Job and only set a TTL when the Job is being marked as completed.
- Write your own controller to manage the cleanup TTL for Jobs that match a particular [selector-selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

## Caveats[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#caveats)

### Updating TTL for finished Jobs[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#updating-ttl-for-finished-jobs)

You can modify the TTL period, e.g. `.spec.ttlSecondsAfterFinished` field of Jobs, after the job is created or has finished. If you extend the TTL period after the existing `ttlSecondsAfterFinished` period has expired, Kubernetes doesn't guarantee to retain that Job, even if an update to extend the TTL returns a successful API response.

### Time skew[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#time-skew)

Because the TTL-after-finished controller uses timestamps stored in the Kubernetes jobs to determine whether the TTL has expired or not, this feature is sensitive to time skew in your cluster, which may cause the control plane to clean up Job objects at the wrong time.

Clocks aren't always correct, but the difference should be very small. Please be aware of this risk when setting a non-zero TTL.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Read [Clean up Jobs automatically](https://kubernetes.io/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically)
    
- Refer to the [Kubernetes Enhancement Proposal](https://github.com/kubernetes/enhancements/blob/master/keps/sig-apps/592-ttl-after-finish/README.md) (KEP) for adding this mechanism.
    

# 7 - CronJob[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-2e4cec01c525b45eccd6010e21cc76d9)

**FEATURE STATE:** `Kubernetes v1.21 [stable]`

A _CronJob_ creates [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) on a repeating schedule.

CronJob is meant for performing regular scheduled actions such as backups, report generation, and so on. One CronJob object is like one line of a _crontab_ (cron table) file on a Unix system. It runs a job periodically on a given schedule, written in [Cron](https://en.wikipedia.org/wiki/Cron) format.

CronJobs have limitations and idiosyncrasies. For example, in certain circumstances, a single CronJob can create multiple concurrent Jobs. See the [limitations](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#cron-job-limitations) below.

When the control plane creates new Jobs and (indirectly) Pods for a CronJob, the `.metadata.name` of the CronJob is part of the basis for naming those Pods. The name of a CronJob must be a valid [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-label-names). Even when the name is a DNS subdomain, the name must be no longer than 52 characters. This is because the CronJob controller will automatically append 11 characters to the name you provide and there is a constraint that the length of a Job name is no more than 63 characters.

## Example[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#example)

This example CronJob manifest prints the current time and a hello message every minute:

[`application/job/cronjob.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/application/job/cronjob.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy application/job/cronjob.yaml to clipboard")

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

([Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/) takes you through this example in more detail).

## Writing a CronJob spec[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-cronjob-spec)

### Schedule syntax[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#schedule-syntax)

The `.spec.schedule` field is required. The value of that field follows the [Cron](https://en.wikipedia.org/wiki/Cron) syntax:

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *
```

For example, `0 0 13 * 5` states that the task must be started every Friday at midnight, as well as on the 13th of each month at midnight.

The format also includes extended "Vixie cron" step values. As explained in the [FreeBSD manual](https://www.freebsd.org/cgi/man.cgi?crontab%285%29):

> Step values can be used in conjunction with ranges. Following a range with `/<number>` specifies skips of the number's value through the range. For example, `0-23/2` can be used in the hours field to specify command execution every other hour (the alternative in the V7 standard is `0,2,4,6,8,10,12,14,16,18,20,22`). Steps are also permitted after an asterisk, so if you want to say "every two hours", just use `*/2`.

**Note:** A question mark (`?`) in the schedule has the same meaning as an asterisk `*`, that is, it stands for any of available value for a given field.

Other than the standard syntax, some macros like `@monthly` can also be used:

|Entry|Description|Equivalent to|
|---|---|---|
|@yearly (or @annually)|Run once a year at midnight of 1 January|0 0 1 1 *|
|@monthly|Run once a month at midnight of the first day of the month|0 0 1 * *|
|@weekly|Run once a week at midnight on Sunday morning|0 0 * * 0|
|@daily (or @midnight)|Run once a day at midnight|0 0 * * *|
|@hourly|Run once an hour at the beginning of the hour|0 * * * *|

To generate CronJob schedule expressions, you can also use web tools like [crontab.guru](https://crontab.guru/).

### Job template[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-template)

The `.spec.jobTemplate` defines a template for the Jobs that the CronJob creates, and it is required. It has exactly the same schema as a [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/), except that it is nested and does not have an `apiVersion` or `kind`. You can specify common metadata for the templated Jobs, such as [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels) or [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations). For information about writing a Job `.spec`, see [Writing a Job Spec](https://kubernetes.io/docs/concepts/workloads/controllers/job/#writing-a-job-spec).

### Deadline for delayed job start[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#starting-deadline)

The `.spec.startingDeadlineSeconds` field is optional. This field defines a deadline (in whole seconds) for starting the Job, if that Job misses its scheduled time for any reason.

After missing the deadline, the CronJob skips that instance of the Job (future occurrences are still scheduled). For example, if you have a backup job that runs twice a day, you might allow it to start up to 8 hours late, but no later, because a backup taken any later wouldn't be useful: you would instead prefer to wait for the next scheduled run.

For Jobs that miss their configured deadline, Kubernetes treats them as failed Jobs. If you don't specify `startingDeadlineSeconds` for a CronJob, the Job occurrences have no deadline.

If the `.spec.startingDeadlineSeconds` field is set (not null), the CronJob controller measures the time between when a job is expected to be created and now. If the difference is higher than that limit, it will skip this execution.

For example, if it is set to `200`, it allows a job to be created for up to 200 seconds after the actual schedule.

### Concurrency policy[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#concurrency-policy)

The `.spec.concurrencyPolicy` field is also optional. It specifies how to treat concurrent executions of a job that is created by this CronJob. The spec may specify only one of the following concurrency policies:

- `Allow` (default): The CronJob allows concurrently running jobs
- `Forbid`: The CronJob does not allow concurrent runs; if it is time for a new job run and the previous job run hasn't finished yet, the CronJob skips the new job run
- `Replace`: If it is time for a new job run and the previous job run hasn't finished yet, the CronJob replaces the currently running job run with a new job run

Note that concurrency policy only applies to the jobs created by the same cron job. If there are multiple CronJobs, their respective jobs are always allowed to run concurrently.

### Schedule suspension[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#schedule-suspension)

You can suspend execution of Jobs for a CronJob, by setting the optional `.spec.suspend` field to true. The field defaults to false.

This setting does _not_ affect Jobs that the CronJob has already started.

If you do set that field to true, all subsequent executions are suspended (they remain scheduled, but the CronJob controller does not start the Jobs to run the tasks) until you unsuspend the CronJob.

**Caution:** Executions that are suspended during their scheduled time count as missed jobs. When `.spec.suspend` changes from `true` to `false` on an existing CronJob without a [starting deadline](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#starting-deadline), the missed jobs are scheduled immediately.

### Jobs history limits[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#jobs-history-limits)

The `.spec.successfulJobsHistoryLimit` and `.spec.failedJobsHistoryLimit` fields are optional. These fields specify how many completed and failed jobs should be kept. By default, they are set to 3 and 1 respectively. Setting a limit to `0` corresponds to keeping none of the corresponding kind of jobs after they finish.

For another way to clean up jobs automatically, see [Clean up finished jobs automatically](https://kubernetes.io/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically).

### Time zones[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#time-zones)

**FEATURE STATE:** `Kubernetes v1.27 [stable]`

For CronJobs with no time zone specified, the [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) interprets schedules relative to its local time zone.

You can specify a time zone for a CronJob by setting `.spec.timeZone` to the name of a valid [time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). For example, setting `.spec.timeZone: "Etc/UTC"` instructs Kubernetes to interpret the schedule relative to Coordinated Universal Time.

A time zone database from the Go standard library is included in the binaries and used as a fallback in case an external database is not available on the system.

## CronJob limitations[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#cron-job-limitations)

### Unsupported TimeZone specification[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#unsupported-timezone-specification)

The implementation of the CronJob API in Kubernetes 1.27 lets you set the `.spec.schedule` field to include a timezone; for example: `CRON_TZ=UTC * * * * *` or `TZ=UTC * * * * *`.

Specifying a timezone that way is **not officially supported** (and never has been).

If you try to set a schedule that includes `TZ` or `CRON_TZ` timezone specification, Kubernetes reports a [warning](https://kubernetes.io/blog/2020/09/03/warnings/) to the client. Future versions of Kubernetes will prevent setting the unofficial timezone mechanism entirely.

### Modifying a CronJob[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#modifying-a-cronjob)

By design, a CronJob contains a template for _new_ Jobs. If you modify an existing CronJob, the changes you make will apply to new Jobs that start to run after your modification is complete. Jobs (and their Pods) that have already started continue to run without changes. That is, the CronJob does _not_ update existing Jobs, even if those remain running.

### Job creation[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job-creation)

A CronJob creates a Job object approximately once per execution time of its schedule. The scheduling is approximate because there are certain circumstances where two Jobs might be created, or no Job might be created. Kubernetes tries to avoid those situations, but does not completely prevent them. Therefore, the Jobs that you define should be _idempotent_.

If `startingDeadlineSeconds` is set to a large value or left unset (the default) and if `concurrencyPolicy` is set to `Allow`, the jobs will always run at least once.

**Caution:** If `startingDeadlineSeconds` is set to a value less than 10 seconds, the CronJob may not be scheduled. This is because the CronJob controller checks things every 10 seconds.

For every CronJob, the CronJob [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) checks how many schedules it missed in the duration from its last scheduled time until now. If there are more than 100 missed schedules, then it does not start the job and logs the error.

```
Cannot determine if job needs to be started. Too many missed start time (> 100). Set or decrease .spec.startingDeadlineSeconds or check clock skew.
```

It is important to note that if the `startingDeadlineSeconds` field is set (not `nil`), the controller counts how many missed jobs occurred from the value of `startingDeadlineSeconds` until now rather than from the last scheduled time until now. For example, if `startingDeadlineSeconds` is `200`, the controller counts how many missed jobs occurred in the last 200 seconds.

A CronJob is counted as missed if it has failed to be created at its scheduled time. For example, if `concurrencyPolicy` is set to `Forbid` and a CronJob was attempted to be scheduled when there was a previous schedule still running, then it would count as missed.

For example, suppose a CronJob is set to schedule a new Job every one minute beginning at `08:30:00`, and its `startingDeadlineSeconds` field is not set. If the CronJob controller happens to be down from `08:29:00` to `10:21:00`, the job will not start as the number of missed jobs which missed their schedule is greater than 100.

To illustrate this concept further, suppose a CronJob is set to schedule a new Job every one minute beginning at `08:30:00`, and its `startingDeadlineSeconds` is set to 200 seconds. If the CronJob controller happens to be down for the same period as the previous example (`08:29:00` to `10:21:00`,) the Job will still start at 10:22:00. This happens as the controller now checks how many missed schedules happened in the last 200 seconds (i.e., 3 missed schedules), rather than from the last scheduled time until now.

The CronJob is only responsible for creating Jobs that match its schedule, and the Job in turn is responsible for the management of the Pods it represents.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn about [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/), two concepts that CronJobs rely upon.
- Read about the detailed [format](https://pkg.go.dev/github.com/robfig/cron/v3#hdr-CRON_Expression_Format) of CronJob `.spec.schedule` fields.
- For instructions on creating and working with CronJobs, and for an example of a CronJob manifest, see [Running automated tasks with CronJobs](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/).
- `CronJob` is part of the Kubernetes REST API. Read the [CronJob](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/cron-job-v1/) API reference for more details.

# 8 - ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pg-27f1331d515d95f76aa1156088b4ad91)

**Note:** A [`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) that configures a [`ReplicaSet`](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is now the recommended way to set up replication.

A _ReplicationController_ ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.

## How a ReplicationController Works[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#how-a-replicationcontroller-works)

If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the ReplicationController starts more pods. Unlike manually created pods, the pods maintained by a ReplicationController are automatically replaced if they fail, are deleted, or are terminated. For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade. For this reason, you should use a ReplicationController even if your application requires only a single pod. A ReplicationController is similar to a process supervisor, but instead of supervising individual processes on a single node, the ReplicationController supervises multiple pods across multiple nodes.

ReplicationController is often abbreviated to "rc" in discussion, and as a shortcut in kubectl commands.

A simple case is to create one ReplicationController object to reliably run one instance of a Pod indefinitely. A more complex use case is to run several identical replicas of a replicated service, such as web servers.

## Running an example ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#running-an-example-replicationcontroller)

This example ReplicationController config runs three copies of the nginx web server.

[`controllers/replication.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/replication.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/replication.yaml to clipboard")

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Run the example job by downloading the example file and then running this command:

```shell
kubectl apply -f https://k8s.io/examples/controllers/replication.yaml
```

The output is similar to this:

```
replicationcontroller/nginx created
```

Check on the status of the ReplicationController using this command:

```shell
kubectl describe replicationcontrollers/nginx
```

The output is similar to this:

```
Name:        nginx
Namespace:   default
Selector:    app=nginx
Labels:      app=nginx
Annotations:    <none>
Replicas:    3 current / 3 desired
Pods Status: 0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=nginx
  Containers:
   nginx:
    Image:              nginx
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen       LastSeen     Count    From                        SubobjectPath    Type      Reason              Message
  ---------       --------     -----    ----                        -------------    ----      ------              -------
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-qrm3m
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-3ntk0
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-4ok8v
```

Here, three pods are created, but none is running yet, perhaps because the image is being pulled. A little later, the same command may show:

```shell
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

To list all the pods that belong to the ReplicationController in a machine readable form, you can use a command like this:

```shell
pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
```

The output is similar to this:

```
nginx-3ntk0 nginx-4ok8v nginx-qrm3m
```

Here, the selector is the same as the selector for the ReplicationController (seen in the `kubectl describe` output), and in a different form in `replication.yaml`. The `--output=jsonpath` option specifies an expression with the name from each pod in the returned list.

## Writing a ReplicationController Manifest[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-a-replicationcontroller-manifest)

As with all other Kubernetes config, a ReplicationController needs `apiVersion`, `kind`, and `metadata` fields.

When the control plane creates new Pods for a ReplicationController, the `.metadata.name` of the ReplicationController is part of the basis for naming those Pods. The name of a ReplicationController must be a valid [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-label-names).

For general information about working with configuration files, see [object management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/).

A ReplicationController also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

### Pod Template[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-template)

The `.spec.template` is the only required field of the `.spec`.

The `.spec.template` is a [pod template](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/), except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a pod template in a ReplicationController must specify appropriate labels and an appropriate restart policy. For labels, make sure not to overlap with other controllers. See [pod selector](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector).

Only a [`.spec.template.spec.restartPolicy`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Always` is allowed, which is the default if not specified.

For local container restarts, ReplicationControllers delegate to an agent on the node, for example the [Kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/).

### Labels on the ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#labels-on-the-replicationcontroller)

The ReplicationController can itself have labels (`.metadata.labels`). Typically, you would set these the same as the `.spec.template.metadata.labels`; if `.metadata.labels` is not specified then it defaults to `.spec.template.metadata.labels`. However, they are allowed to be different, and the `.metadata.labels` do not affect the behavior of the ReplicationController.

### Pod Selector[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#pod-selector)

The `.spec.selector` field is a [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors). A ReplicationController manages all the pods with labels that match the selector. It does not distinguish between pods that it created or deleted and pods that another person or process created or deleted. This allows the ReplicationController to be replaced without affecting the running pods.

If specified, the `.spec.template.metadata.labels` must be equal to the `.spec.selector`, or it will be rejected by the API. If `.spec.selector` is unspecified, it will be defaulted to `.spec.template.metadata.labels`.

Also you should not normally create any pods whose labels match this selector, either directly, with another ReplicationController, or with another controller such as Job. If you do so, the ReplicationController thinks that it created the other pods. Kubernetes does not stop you from doing this.

If you do end up with multiple controllers that have overlapping selectors, you will have to manage the deletion yourself (see [below](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#working-with-replicationcontrollers)).

### Multiple Replicas[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#multiple-replicas)

You can specify how many pods should run concurrently by setting `.spec.replicas` to the number of pods you would like to have running concurrently. The number running at any time may be higher or lower, such as if the replicas were just increased or decreased, or if a pod is gracefully shutdown, and a replacement starts early.

If you do not specify `.spec.replicas`, then it defaults to 1.

## Working with ReplicationControllers[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#working-with-replicationcontrollers)

### Deleting a ReplicationController and its Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deleting-a-replicationcontroller-and-its-pods)

To delete a ReplicationController and all its pods, use [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete). Kubectl will scale the ReplicationController to zero and wait for it to delete each pod before deleting the ReplicationController itself. If this kubectl command is interrupted, it can be restarted.

When using the REST API or [client library](https://kubernetes.io/docs/reference/using-api/client-libraries), you need to do the steps explicitly (scale replicas to 0, wait for pod deletions, then delete the ReplicationController).

### Deleting only a ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deleting-only-a-replicationcontroller)

You can delete a ReplicationController without affecting any of its pods.

Using kubectl, specify the `--cascade=orphan` option to [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete).

When using the REST API or [client library](https://kubernetes.io/docs/reference/using-api/client-libraries), you can delete the ReplicationController object.

Once the original is deleted, you can create a new ReplicationController to replace it. As long as the old and new `.spec.selector` are the same, then the new one will adopt the old pods. However, it will not make any effort to make existing pods match a new, different pod template. To update pods to a new spec in a controlled way, use a [rolling update](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-updates).

### Isolating pods from a ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#isolating-pods-from-a-replicationcontroller)

Pods may be removed from a ReplicationController's target set by changing their labels. This technique may be used to remove pods from service for debugging and data recovery. Pods that are removed in this way will be replaced automatically (assuming that the number of replicas is not also changed).

## Common usage patterns[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#common-usage-patterns)

### Rescheduling[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rescheduling)

As mentioned above, whether you have 1 pod you want to keep running, or 1000, a ReplicationController will ensure that the specified number of pods exists, even in the event of node failure or pod termination (for example, due to an action by another control agent).

### Scaling[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#scaling)

The ReplicationController enables scaling the number of replicas up or down, either manually or by an auto-scaling control agent, by updating the `replicas` field.

### Rolling updates[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#rolling-updates)

The ReplicationController is designed to facilitate rolling updates to a service by replacing pods one-by-one.

As explained in [#1353](https://issue.k8s.io/1353), the recommended approach is to create a new ReplicationController with 1 replica, scale the new (+1) and old (-1) controllers one by one, and then delete the old controller after it reaches 0 replicas. This predictably updates the set of pods regardless of unexpected failures.

Ideally, the rolling update controller would take application readiness into account, and would ensure that a sufficient number of pods were productively serving at any given time.

The two ReplicationControllers would need to create pods with at least one differentiating label, such as the image tag of the primary container of the pod, since it is typically image updates that motivate rolling updates.

### Multiple release tracks[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#multiple-release-tracks)

In addition to running multiple releases of an application while a rolling update is in progress, it's common to run multiple releases for an extended period of time, or even continuously, using multiple release tracks. The tracks would be differentiated by labels.

For instance, a service might target all pods with `tier in (frontend), environment in (prod)`. Now say you have 10 replicated pods that make up this tier. But you want to be able to 'canary' a new version of this component. You could set up a ReplicationController with `replicas` set to 9 for the bulk of the replicas, with labels `tier=frontend, environment=prod, track=stable`, and another ReplicationController with `replicas` set to 1 for the canary, with labels `tier=frontend, environment=prod, track=canary`. Now the service is covering both the canary and non-canary pods. But you can mess with the ReplicationControllers separately to test things out, monitor the results, etc.

### Using ReplicationControllers with Services[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#using-replicationcontrollers-with-services)

Multiple ReplicationControllers can sit behind a single service, so that, for example, some traffic goes to the old version, and some goes to the new version.

A ReplicationController will never terminate on its own, but it isn't expected to be as long-lived as services. Services may be composed of pods controlled by multiple ReplicationControllers, and it is expected that many ReplicationControllers may be created and destroyed over the lifetime of a service (for instance, to perform an update of pods that run the service). Both services themselves and their clients should remain oblivious to the ReplicationControllers that maintain the pods of the services.

## Writing programs for Replication[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#writing-programs-for-replication)

Pods created by a ReplicationController are intended to be fungible and semantically identical, though their configurations may become heterogeneous over time. This is an obvious fit for replicated stateless servers, but ReplicationControllers can also be used to maintain availability of master-elected, sharded, and worker-pool applications. Such applications should use dynamic work assignment mechanisms, such as the [RabbitMQ work queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html), as opposed to static/one-time customization of the configuration of each pod, which is considered an anti-pattern. Any pod customization performed, such as vertical auto-sizing of resources (for example, cpu or memory), should be performed by another online controller process, not unlike the ReplicationController itself.

## Responsibilities of the ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#responsibilities-of-the-replicationcontroller)

The ReplicationController ensures that the desired number of pods matches its label selector and are operational. Currently, only terminated pods are excluded from its count. In the future, [readiness](https://issue.k8s.io/620) and other information available from the system may be taken into account, we may add more controls over the replacement policy, and we plan to emit events that could be used by external clients to implement arbitrarily sophisticated replacement and/or scale-down policies.

The ReplicationController is forever constrained to this narrow responsibility. It itself will not perform readiness nor liveness probes. Rather than performing auto-scaling, it is intended to be controlled by an external auto-scaler (as discussed in [#492](https://issue.k8s.io/492)), which would change its `replicas` field. We will not add scheduling policies (for example, [spreading](https://issue.k8s.io/367#issuecomment-48428019)) to the ReplicationController. Nor should it verify that the pods controlled match the currently specified template, as that would obstruct auto-sizing and other automated processes. Similarly, completion deadlines, ordering dependencies, configuration expansion, and other features belong elsewhere. We even plan to factor out the mechanism for bulk pod creation ([#170](https://issue.k8s.io/170)).

The ReplicationController is intended to be a composable building-block primitive. We expect higher-level APIs and/or tools to be built on top of it and other complementary primitives for user convenience in the future. The "macro" operations currently supported by kubectl (run, scale) are proof-of-concept examples of this. For instance, we could imagine something like [Asgard](https://netflixtechblog.com/asgard-web-based-cloud-management-and-deployment-2c9fc4e4d3a1) managing ReplicationControllers, auto-scalers, services, scheduling policies, canaries, etc.

## API Object[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#api-object)

Replication controller is a top-level resource in the Kubernetes REST API. More details about the API object can be found at: [ReplicationController API object](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#replicationcontroller-v1-core).

## Alternatives to ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#alternatives-to-replicationcontroller)

### ReplicaSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#replicaset)

[`ReplicaSet`](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is the next-generation ReplicationController that supports the new [set-based label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement). It's mainly used by [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) as a mechanism to orchestrate pod creation, deletion and updates. Note that we recommend using Deployments instead of directly using Replica Sets, unless you require custom update orchestration or don't require updates at all.

### Deployment (Recommended)[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#deployment-recommended)

[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a higher-level API object that updates its underlying Replica Sets and their Pods. Deployments are recommended if you want the rolling update functionality, because they are declarative, server-side, and have additional features.

### Bare Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#bare-pods)

Unlike in the case where a user directly created pods, a ReplicationController replaces pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicationController even if your application requires only a single pod. Think of it similarly to a process supervisor, only it supervises multiple pods across multiple nodes instead of individual processes on a single node. A ReplicationController delegates local container restarts to some agent on the node, such as the kubelet.

### Job[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#job)

Use a [`Job`](https://kubernetes.io/docs/concepts/workloads/controllers/job/) instead of a ReplicationController for pods that are expected to terminate on their own (that is, batch jobs).

### DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#daemonset)

Use a [`DaemonSet`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicationController for pods that provide a machine-level function, such as machine monitoring or machine logging. These pods have a lifetime that is tied to a machine lifetime: the pod needs to be running on the machine before other pods start, and are safe to terminate when the machine is otherwise ready to be rebooted/shutdown.

## What's next[](https://kubernetes.io/docs/concepts/workloads/controllers/_print/#what-s-next)

- Learn about [Pods](https://kubernetes.io/docs/concepts/workloads/pods).
- Learn about [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), the replacement for ReplicationController.
- `ReplicationController` is part of the Kubernetes REST API. Read the [ReplicationController](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/replication-controller-v1/) object definition to understand the API for replication controllers.

[Home](https://kubernetes.io/docs/home/)[Blog](https://kubernetes.io/blog/)[Training](https://kubernetes.io/training/)[Partners](https://kubernetes.io/partners/)[Community](https://kubernetes.io/community/)[Case Studies](https://kubernetes.io/case-studies/)

- [](https://discuss.kubernetes.io/)
- [](https://twitter.com/kubernetesio)
- [](https://calendar.google.com/calendar/embed?src=calendar%40kubernetes.io)
- [](https://youtube.com/kubernetescommunity)

- [](https://github.com/kubernetes/kubernetes)
- [](https://slack.k8s.io/)
- [](https://git.k8s.io/community/contributors/guide)
- [](https://stackoverflow.com/questions/tagged/kubernetes)

© 2023 The Kubernetes Authors | Documentation Distributed under [CC BY 4.0](https://git.k8s.io/website/LICENSE)  
Copyright © 2023 The Linux Foundation ®. All rights reserved. The Linux Foundation has registered trademarks and uses trademarks. For a list of trademarks of The Linux Foundation, please see our [Trademark Usage page](https://www.linuxfoundation.org/trademark-usage)  
ICP license: 京ICP备17074266号-3