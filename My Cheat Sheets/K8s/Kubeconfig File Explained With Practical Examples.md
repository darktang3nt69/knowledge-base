# Kubeconfig File Explained With Practical Examples

- by[Bibin Wilson](https://devopscube.com/author/bibinwilson/ "View all posts by Bibin Wilson")
- April 24, 2023

In this blog, you will learn how to **connect to a kubernetes cluster using the Kubeconfig** file using different methods. Also, you will learn to generate a custom Kubeconfig file.

**Table of Contents**  [show](https://devopscube.com/kubernetes-kubeconfig-file/#) 

## What is a Kubeconfig file?

A **Kubeconfig is a YAML file** with all the [Kubernetes cluster](https://devopscube.com/setup-kubernetes-cluster-kubeadm/) details, certificates, and secret token to authenticate the cluster. You might get this config file directly from the cluster administrator or from a cloud platform if you are using managed Kubernetes cluster.

When you use `kubectl`, it uses the information in the kubeconfig file to connect to the kubernetes cluster API. The default location of the Kubeconfig file is `$HOME/.kube/config`

Also, [kubernetes cluster components](https://devopscube.com/kubernetes-architecture-explained/) like controller manager, scheduler and kubelet use the kubeconfig files to interact with the API server.

## Example Kubeconfig File

Here is an example of a Kubeconfig. It needs the following key information to connect to the Kubernetes clusters.

1. **certificate-authority-data**: Cluster CA
2. **server**: Cluster endpoint (IP/DNS of the master node)
3. **name**: Cluster name
4. **user**: name of the user/service account.
5. **token**: Secret token of the user/service account.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <ca-data-here>
    server: https://your-k8s-cluster.com
  name: <cluster-name>
contexts:
- context:
    cluster:  <cluster-name>
    user:  <cluster-name-user>
  name:  <cluster-name>
current-context:  <cluster-name>
kind: Config
preferences: {}
users:
- name:  <cluster-name-user>
  user:
    token: <secret-token-here>
```

## Different Methods to Connect Kubernetes Cluster With Kubeconfig File

You can use the Kubeconfig in different ways and each way has its own precedence. Here is the precedence in order.

1. **Kubectl Context:** Kubeconfig with kubectl overrides all other configs. It has the highest precedence.
2. **Environment Variable:** KUBECONFIG env variable overrides the current context.
3. **Command-Line Reference:** The current context has the least precedence over the inline config reference and env variable.

Now let’s take a look at all **three ways to use the Kubeconfig file.**

### Method 1: Connect to Kubernetes Cluster With Kubeconfig Kubectl Context

To connect to the Kubernetes cluster, the basic prerequisite is the Kubectl CLI plugin. If you don’t have the CLI installed, follow the [instructions given here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).

Now follow the steps given below to use the kubeconfig file to interact with the cluster.

#### Step 1: Move kubeconfig to .kube directory.

Kubectl interacts with the kubernetes cluster using the details available in the Kubeconfig file. By default, kubectl looks for the config file in the `/.kube` location.

Let’s move the kubeconfig file to the .kube directory. Replace `/path/to/kubeconfig` with your kubeconfig current path.

```bash
mv /path/to/kubeconfig ~/.kube
```

#### Step 2: List all cluster contexts

You can have any number of kubeconfig in the `.kube` directory. Each config will have a unique context name (ie, the name of the cluster). You can validate the Kubeconfig file by listing the contexts. You can list all the contexts using the following command. It will list the context name as the name of the cluster.

```bash
kubectl config get-contexts
```

#### Step 3: Set the current context

Now you need to set the current context to your kubeconfig file. You can set that using the following command. replace `<cluster-name>` with your listed context name.

```bash
kubectl config use-context <cluster-name>  
```

For example,

```bash
kubectl config use-context my-dev-cluster
```

#### Step 4: Validate the Kubernetes cluster connectivity

To validate the cluster connectivity, you can execute the following kubectl command to list the cluster nodes.

```bash
kubectl get nodes
```

### Method 2: Connect with KUBECONFIG environment variable

You can set the `KUBECONFIG` environment variable with the `kubeconfig` file path to connect to the cluster. So wherever you are using the kubectl command from the terminal, the `KUBECONFIG` env variable should be available. If you set this variable, it overrides the current cluster context.

You can set the variable using the following command. Where `dev_cluster_config` is the `kubeconfig` file name.

```bash
KUBECONFIG=$HOME/.kube/dev_cluster_config
```

### Method 3: Using Kubeconfig File With Kubectl

You can pass the Kubeconfig file with the Kubectl command to override the current context and KUBECONFIG env variable.

Here is an example to get nodes.

```bash
kubectl get nodes --kubeconfig=$HOME/.kube/dev_cluster_config
```

Also you can use,

```bash
KUBECONFIG=$HOME/.kube/dev_cluster_config kubectl get nodes
```

## Merging Multiple Kubeconfig Files

Usually, when you work with Kubernetes services like GKE, all the cluster contexts get added as a single file. However, there are situations where you will be given a Kubeconfig file with limited access to connect to prod or non-prod servers. To manage all clusters effectively using a single config, you can merge the other Kubeconfig files to the default `$HOME/.kube/config` file using the supported kubectl command.

Lets assume you have three Kubeconfig files in the `$HOME/.kube/` directory.

1. config (default kubeconfig)
2. dev_config
3. test_config

You can merge all the three configs into a single file using the following command. Ensure you are running the command from the `$HOME/.kub`e directory

```bash
KUBECONFIG=config:dev_config:test_config kubectl config view --merge --flatten > config.new
```

The above command creates a merged config named `config.new`.

Now rename the old `**$HOME.kube/config**` file.

```bash
 mv $HOME/.kube/config $HOME/.kube/config.old
```

Rename the `config.new` to config.

```bash
mv $HOME/.kube/config.new $HOME/.kube/config
```

To verify the configuration, try listing the contexts from the config.

```bash
kubectl config get-contexts
```

If you want to view the kubeconfig file in a condensed format to analyze all the configurations, you can use the following minify command.

```bash
kubectl config view --minify
```

## How to Generate Kubeconfig File?

Now we will look at creating Kubeconfig files using the serviceaccount method. serviceaccount is the default user type managed by Kubernetes API.

A kubeconfig needs the following important details.

1. Cluster endpoint (IP or DNS name of the cluster)
2. Cluster CA Certificate
3. Cluster name
4. Service account user name
5. Service account token

> **Note**: To generate a Kubeconfig file, you need to have admin permissions in the cluster to create service accounts and roles.

For this demo, I am creating a service account with `clusterRole` that has limited access to cluster-wide resources. You can also create a normal role and Rolebinding that limits the user’s access to a specific namespace.

#### Step 1: Create a Service Account

The service account name will be the user name in Kubeconfig. Here I am creating the service account in the `kube-system` as I am creating a clusterRole. If you want to create a config to give namespace level limited access, create the service account in the required namespace.

```bash
kubectl -n kube-system create serviceaccount devops-cluster-admin
```

#### Step 2: Create a Secret Object for the Service Account

From Kubernetes Version 1.24, the secret for the service account has to be created separately with an annotation `kubernetes.io/service-account.name` and type `kubernetes.io/service-account-token`

Let’s create a secret named **devops-cluster-admin-secret** with the annotation and type.

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: devops-cluster-admin-secret
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: devops-cluster-admin
type: kubernetes.io/service-account-token
EOF
```

#### Step 3: Create a ClusterRole

Let’s create a `clusterRole` with limited privileges to cluster objects. You can add the required object access as per your requirements. Refer to the s[ervice account with clusterRole access](https://devopscube.com/kubernetes-api-access-service-account/) blog for more information.

If you want to create a namespace-scoped role, refer to [creating service account with role](https://devopscube.com/create-kubernetes-role/).

Execute the following command to create the clusterRole.

```bash
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: devops-cluster-admin
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
EOF
```

#### Step 4: Create ClusterRoleBinding

The following YAML is a ClusterRoleBinding that binds the `devops-cluster-admin` service account with the `devops-cluster-admin` clusterRole.

```bash
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devops-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: devops-cluster-admin
subjects:
- kind: ServiceAccount
  name: devops-cluster-admin
  namespace: kube-system
EOF
```

#### Step 5: Get all Cluster Details & Secrets

We will retrieve all the required kubeconfig details and save them in variables. Then, finally, we will substitute it directly with the Kubeconfig YAML.

If you have used a different secret name, replace **`devops-cluster-admin-secret`** with your secret name,

```bash
export SA_SECRET_TOKEN=$(kubectl -n kube-system get secret/devops-cluster-admin-secret -o=go-template='{{.data.token}}' | base64 --decode)

export CLUSTER_NAME=$(kubectl config current-context)

export CURRENT_CLUSTER=$(kubectl config view --raw -o=go-template='{{range .contexts}}{{if eq .name "'''${CLUSTER_NAME}'''"}}{{ index .context "cluster" }}{{end}}{{end}}')

export CLUSTER_CA_CERT=$(kubectl config view --raw -o=go-template='{{range .clusters}}{{if eq .name "'''${CURRENT_CLUSTER}'''"}}"{{with index .cluster "certificate-authority-data" }}{{.}}{{end}}"{{ end }}{{ end }}')

export CLUSTER_ENDPOINT=$(kubectl config view --raw -o=go-template='{{range .clusters}}{{if eq .name "'''${CURRENT_CLUSTER}'''"}}{{ .cluster.server }}{{end}}{{ end }}')
```

#### Step 6: Generate the Kubeconfig With the variables.

If you execute the following YAML, all the variables get substituted and a config named `devops-cluster-admin-config` gets generated.

```bash
cat << EOF > devops-cluster-admin-config
apiVersion: v1
kind: Config
current-context: ${CLUSTER_NAME}
contexts:
- name: ${CLUSTER_NAME}
  context:
    cluster: ${CLUSTER_NAME}
    user: devops-cluster-admin
clusters:
- name: ${CLUSTER_NAME}
  cluster:
    certificate-authority-data: ${CLUSTER_CA_CERT}
    server: ${CLUSTER_ENDPOINT}
users:
- name: devops-cluster-admin
  user:
    token: ${SA_SECRET_TOKEN}
EOF
```

#### Step 7: Validate the generated Kubeconfig

To validate the Kubeconfig, execute it with the kubectl command to see if the cluster is getting authenticated.

```bash
kubectl get nodes --kubeconfig=devops-cluster-admin-config 
```

> **Note**: In cloud environments, cluster RBAC (Role-Based Access Control) can be mapped with normal IAM (Identity and Access Management) users. This allows organizations to control access to the cluster based on IAM policies, which can be used to create restrictive kubeconfig files. Additionally, other services, such as [OIDC](https://openid.net/connect/) (OpenID Connect), can be used to manage users and create kubeconfig files that limit access to the cluster based on specific security requirements.

## Kubeconfig File FAQs

Let’s look at some of the frequently asked Kubeconfig file questions.

### Where do I put the Kubeconfig file?

The default Kubeconfig file location is `` `**$HOME/.kube/**` `` folder in the home directory. Kubectl looks for the kubeconfig file using the conext name from the .`kube` folder. However, if you are using the `KUBECONFIG` environment variable, you can place the kubeconfig file in a preferred folder and refer to the path in the `KUBECONFIG` environment variable.

### Where is the Kubeconfig file located?

All the kubeconfig files are located in the .kube directory in the user home directory. That is `**$HOME/.kube/config**`

### How to manage multiple Kubeconfig files?

You can store all the kubeconfig files in `$HOME/.kube` directory. You need to change the cluster context to connect to a specific cluster.

### How to create a Kubeconfig file?

To create a Kubeconfig file, you need to have the cluster endpoint details, cluster CA certificate, and authentication token. Then you need to create a Kubernetes YAML object of type config with all the cluster details.

### How to use Proxy with Kubeconfig

If you are behind a corporate proxy, you can use `**proxy-url**: https://proxy.host:port` in your Kubeconfig file to connect to the cluster.