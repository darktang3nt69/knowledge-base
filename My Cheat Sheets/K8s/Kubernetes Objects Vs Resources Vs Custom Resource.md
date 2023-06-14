# Kubernetes Objects Vs Resources Vs Custom Resource

- by[Bibin Wilson](https://devopscube.com/author/bibinwilson/ "View all posts by Bibin Wilson")
- March 17, 2023

In this blog you will learn about Kubernetes objects, resources, custom resources and their differences in detail.

While learning Kubernetes, you will quite often come across the names “Kubernetes Object and “Kubernetes resources”. Lets understand what do they really mean.

## What is Kubernetes Object?

In Kubernetes, any item that a user creates and retains is referred to as an “Object.” These can include various items such as namespaces, pods, [deployments](https://devopscube.com/kubernetes-deployment-tutorial/), volumes, or secrets.

If you look at the [Kubernetes architecture](https://devopscube.com/kubernetes-architecture-explained/), the etcd component plays an important role in persisting cluster objects amongst other things. Any object you create, gets stored in etcd.

[etcd](https://devopscube.com/backup-etcd-restore-kubernetes/) stores all objects under the key named `**/registry**` directory. For instance, details regarding a pod called Nginx in the default namespace can be retrieved by searching for **`/registry/pods/default/nginx`**. The following image shows the registry paths where different objects are stored.

[![Kubernetes objects stored in ecd.](https://devopscube.com/wp-content/uploads/2023/02/image-21.png)](https://devopscube.com/wp-content/uploads/2023/02/image-21.png)

To create these objects, you describe what you want (desired state) in a file using either YAML or JSON. It is called a Object Specification.

Here is an example of a Pod Object Specification (highlighted in bold) where we describe the desired stated of the pod.

```
# Pod spec
apiVersion: v1
kind: Pod
metadata:
  name: webserver-pod
spec:
  containers:
  - name: webserver
    image: nginx:latest
    ports:
    - containerPort: 80
```

Once you’ve created an object, you can ask Kubernetes for information about it using Kubectl or API calls to the API server.

## Common Object Parameters

There are many object types in Kubernetes, however, there are common parameters for every object.

Following are the common set of parameters shared by Kubernetes objects

|Parameter|Description|
|---|---|
|**apiVersion**|The Kubernetes API version for the object.  <br>– Alpha  <br>– Beta  <br>– Stable|
|**kind**|The type of object being defined  <br>– Pod  <br>– Deployment  <br>– Service  <br>– Configmap, etc.|
|**metadata**|metadata is used to uniquely identify and describe a Kubernetes object. Here are some of the common key metadata that can be added to an object  <br>– labels  <br>– name  <br>– namespace  <br>– annotations  <br>– finalizers etc|
|**spec**|Under the ‘spec’ section of a Kubernetes object definition, we declare the desired state and characteristics of the object that we want to create.|

## What are Kubernetes Resources?

In Kubernetes, everything is accessed through APIs.

To create different types of objects such as pods, namespaces, configmaps etc, Kubernetes API server provides API endpoints.

These object-specific endpoints are called **API resources or resources**. For example, the API endpoint used to create a pod is referred to as a **Pod resource**.

In simpler terms, a resource is a **specific API URL** used to access an object, and they can be accessed through HTTP verbs such as GET, POST, and DELETE.

For instance, the `**/api/v1/pods**` resource can be used to retrieve a list of Pod objects, and an individual Pod object can be obtained from the `**/api/v1/namespaces/pods/**` resource.

Here are some examples of API resource endpoints in Kubernetes:

1. `/api/v1/namespaces`: This API resource retrieves a list of all namespaces in the cluster.
2. `/api/v1/pods`: This API resource retrieves a list of all pods in the cluster.
3. `/api/v1/pods/{pod-name}`: This API resource retrieves information about a specific pod, where `{pod-name}` is the name of the pod.
4. `/api/v1/namespaces/{namespace-name}/pods`: This API resource retrieves a list of all pods in a specific namespace, where `{namespace-name}` is the name of the namespace.
5. `/api/v1/namespaces/{namespace-name}/pods/{pod-name}`: This API resource retrieves information about a specific pod in a specific namespace.

These endpoints are just a few examples of the many API resources available.

When creating a Kubernetes object using kubectl, the **YAML specification is converted to JSON format** and sent to the Pod resource (Pod API endpoint).

It’s important to note that the terms pods, volumes, and other similar terms are referred to as generic “resources,” not API resources, so it’s important not to confuse the two.

The following image shows how a Kubernetes object spec is send to the API servers pod API resource via kubectl to be persisted in etcd.

[![Kubernetes Objects Vs API Resources](https://devopscube.com/wp-content/uploads/2023/02/image-20.png)](https://devopscube.com/wp-content/uploads/2023/02/image-20.png)

## What are Kubernetes Custom Resources?

Kubernetes comes with a native list of API resources such as pods, deployments, secrets, configmaps, and more.

For example, if you deploy a pod object, it creates a pod with specified containers.

But what if you want to **create your own API resource?**

Let’s say you need a resource named “**backup**” and if you deploy the backup object, it should perform a etcd backup and uploads it to S3.

Luckily, Kubernetes is highly extensible and allows users to create their own resources and integrate them with core Kubernetes APIs. If you develop your own custom API in Kubernetes to perform a specific action, you **can call it a custom resource**.

> **Note**: To create a custom resources, you need to implement a controller that handles the custom resource.

Here is how a custom object specification for backup would look like. When you deploy this object, the custom controller the user developed performs the desired action at the backend.

```
apiVersion: devopscube.com/v1
kind: Backup
metadata:
  name: my-backup
spec:
  etcdEndpoint: http://etcd:2379
  s3Bucket: my-bucket
  s3Region: us-west-2
```

## Kubenetes Manifests

Normally we call [Kubernetes YAML](https://devopscube.com/create-kubernetes-yaml/) files as manifests. It includes specifications for one or more Kubernetes objects, such as Deployments, Services, ConfigMaps, Secrets etc.

The following image shows an example of a Kubernetes manifest file where we have a Pod and Sevice object defined. If you run this manifest it will create both the pod and service object.

[![image 14](https://devopscube.com/wp-content/uploads/2023/03/image-14-791x1024.png)](https://devopscube.com/wp-content/uploads/2023/03/image-14.png)

Is there a maximum limit for the number of objects in a manifest file?

No, but there is a default size limit of 1MB. However, it is recommended to keep the number of objects per file to a manageable level for ease of maintenance and readability if you choose to use manifest files.