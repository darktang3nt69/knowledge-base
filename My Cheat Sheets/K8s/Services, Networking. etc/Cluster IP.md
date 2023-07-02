In Kubernetes, the Cluster IP is an internal IP address assigned to a Service within a cluster. It is used to expose the Service to other resources within the same cluster.

A Cluster IP is automatically assigned to a Service when it is created. It is a virtual IP address that represents the Service internally within the cluster. Each Service in a Kubernetes cluster is assigned a unique Cluster IP.

The Cluster IP is primarily used for communication between Services within the cluster. Other resources, such as Pods or other Services, can use the Cluster IP to access the Service. However, the Cluster IP is not accessible from outside the cluster or from other networks.

The Cluster IP is a key component of Kubernetes networking. It enables Services to be decoupled from the underlying Pods and provides a stable and consistent endpoint for accessing the Service within the cluster.

It's important to note that the Cluster IP is specific to the Service and does not change when Pods are added, removed, or rescheduled within the cluster.