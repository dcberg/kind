# Local Kind Kuberentes Environment
It's important to be able to development and test against a Kubneretes cluster which you can setup and manage locally on your laptop. My personal favorite tool is to use [Kind](https://kind.sigs.k8s.io/) to setup a multi-node Kubernetes cluster locally. This repository contains documentation and resources to help one construct a Kind environment that actually works on their laptop including proper ingress support. 

Note, the instructions in this respository assume you are using a Mac for development. Variations may exist if you use Linux or Windows.

# Prerequisites
There are some tools that you will need to setup prior to configuring your Kind environment.

## Docker Desktop
Kind runs Kubernetes nodes as containers; therefore, you must have [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac) installed. 

Follow the instructions [here](https://hub.docker.com/editions/community/docker-ce-desktop-mac) to install Docker Desktop for Mac. Nothing difficult, just follow the instructions.

### Configure Docker Desktop
You will need to adjust the settings of your Docker Desktop to ensure that the kubernetes clusters work well. 

Go ahead and start Docker Desktop if you haven't already.

From the menu select **Preferences**.

Go to the **Resources** section to adjust the CPU and Memory allocated to containers from your laptop. I would recommend adjusting the values to be at least 4 cores for CPUs and 8GB for Memory. You can go higher if you wish but these values should be sufficient for most local development needs.

![Docker Desktop Preferences](/images/docker-preferences.png)

Click **Apply & Restart** button.


## Kubernetes CLI
You must have the Kubernetes CLI installed. Not absolutely necessary to run Kind but, let's face it, you will be using Kubernetes so you need the CLI installed.

The simplest approach to install the k8s CLI on a mac is via homebrew.

```
brew install kubectl
```
Other CLI installation options are available [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

## Kind
You will obviously need Kind installed as well.

```
brew install kind
```
Addtional installation options can be found [here](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).

## Setup Lens
Lens is a nice graphical tool for accessing, inspecting, and debugging your clusters. Lens exercises kubectl commands under the covers but it allows novice users to easily explore and access information about resources in the cluster. Lens also allows expert users to be more efficient when working with clusters.

You can install Lens directly from the [Lens Development site](https://k8slens.dev/). Simply click on the **Download** button and select the archive for your operating system. Unzip the archive and execute the install to have Lens installed on your system.

# Create a multi-node cluster
I'm going to walk you through creating a simple three node cluster with a non-HA control node.
Kind uses a simple YAML file to describe the cluster. I'm not going to walk you through every setting but I will explain some key points that you will want to use.

First ensure that you clone this repository and `cd` into the root directory from a terminal session.

```
git clone git@github.com:dcberg/kind.git
```

You can kick off the creation now and then read about the key points in the sections that follow.

```
kind create cluster --name test --config scripts/cluster-3.yaml
```
This script will create a three node Kube 1.19 cluster. There is a single control node identified by the `control-plane` role and the version is controlled by the container image that is used. In this case the 1.19 node image is used. You can find the full list of container node images in the [release notes](https://github.com/kubernetes-sigs/kind/releases).

```
- role: control-plane
  image: kindest/node:v1.19.1@sha256:98cf5288864662e37115e362b23e4369c8c4a408f99cbc06e58ac30ddc721600
```

The worker nodes are exactly the same as the control-plane node except with the role set to `worker`.

To allow ingress to work within a local Kind cluster, it is necessary to select one of the nodes to host the ingress controller. On Mac and Windows machines, Docker Desktop does not bridge the container network with the host. Thus you cannot use the node IP directly from the host (i.e., your laptop). In the config yaml above, I have selected the first worker node to be available for ingress. I adjusted the worker node to inject a label (`ingress-ready=true`) which will be used by a node-selector when deploying the ingress-controller. The other adjustment is to set `extraPortMappings` to map the container 80 and 443 ports to the 80 and 443 ports on the host (i.e., your laptop). Note, only one node can map to the ports on the host. It is also necessary to set the `listenAddress` to be `127.0.0.1` for the local. This combination of the host port maps and the localhost listenAddress allows the ingress to be addressed directly from your local machine.

```
- role: worker
  image: kindest/node:v1.19.1@sha256:98cf5288864662e37115e362b23e4369c8c4a408f99cbc06e58ac30ddc721600
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: "127.0.0.1"
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    listenAddress: "127.0.0.1"
    protocol: TCP
 ```
You can validate that the cluster was created successfully.

```
kind get clusters
```
will list the `test` cluster that you just created. You can get the list of nodes for the cluster as well.

```
kind get nodes --name test
```

```
test-worker2
test-control-plane
test-worker3
test-worker
```

After creating a Kind cluster, the kubernetes context is automatically set to the new cluster. You can confirm this by running the following command.

```
kubectl cluster-info --context kind-test
```
You should see a result similar to mine.

```
Kubernetes control plane is running at https://127.0.0.1:53759
KubeDNS is running at https://127.0.0.1:53759/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

You can now run kubectl commands. Try getting the nodes.

```
kubectl get nodes
```

```
NAME                 STATUS   ROLES    AGE     VERSION
test-control-plane   Ready    master   9m11s   v1.19.1
test-worker          Ready    <none>   8m40s   v1.19.1
test-worker2         Ready    <none>   8m40s   v1.19.1
test-worker3         Ready    <none>   8m40s   v1.19.1
```

# Access your cluster with Lens
Launch Lens if you haven't already. Lens will open with no clusters connected. 

## Configure connection to cluster
Click on the **+** button to configure a new connection to your kind cluster.

By default Lens will read your system kubeconfig file and automatically detect the new **kind-test** cluster that you created. If you have additional clusters on your system, you may need to select the **kind-test** context from the drop down menu.

![Lens cluster connection](/images/lens-cluster-connection.png)

Lens will now connect to your cluster and you can explore a bit to ensure that resource information is properly being read (e.g., click on **Nodes** to see the three worker nodes).

## Configure metrics collection
If you select on the **Cluster** section in the left menu you will see messages indicating that metrics are not available because Prometheus has not been setup. You could manually setup Prometheus yourself or you can let Lens do it for you. Having metrics available directly in Lens is helpful for debugging memory leaks as well as setting up proper resource requests and limits for your containers.

Right click on the image of your cluster in the far left navigation and select **Settings**

Scroll to the bottom of the Settings page to the *Features* section and click on the **Install** button for the *Metrics Stack*. This will setup a prometheus instance within your cluster.

![Lens prometheus install](/images/lens-prometheus.png)

You will need to wait a few minutes while the install is completing and for metrics to be gathered for the cluster. Close the settings page. You should now be on the main Cluster page and you should start to see cluster wide metrics.

![Cluster metrics](/images/lens-cluster-metrics.png)

If you navigate to the **Pods** section under **Workloads** you will see metrics per pod as well.

![Pod memory](/images/lens-pod-memory.png)

## Install and Configure Ingress
You now have a basic three node cluster with metrics gathering. Now we want to be able to deploy and access services. We will do that through an [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) in the cluster. We will be using the kubernetes community nginx ingress controller.

From your terminal session, execute the following command to install the nginx ingress controller into your cluster.

```
kubectl apply --filename https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml

kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```
The config file is specifically designed for kind clusters to ensure that the ingress controller is deployed onto the node that you have labelled as `ingress-ready=true`. This is important within the kind environment and Docker Desktop so that we can access the services from your local host.

As a side note, Lens will update showing you the deployments that are taking place. For example you can see the pending deployments on the main **Workloads** page.

![Lens workloads](/images/lens-ingress-deploy.png)

Let's deploy a simple hello-world service into the cluster for testing purposes.

```
kubectl run hello \
  --expose \
  --image nginxdemos/hello:plain-text \
  --port 80
```
We will now configure an ingress resource that will allow us to access the hello service using a URL and our nginx ingress controller.

```
k apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
spec:
  rules:
    - host: hello.test.com
      http:
        paths:
          - pathType: ImplementationSpecific
            backend:
              service:
                name: hello
                port:
                  number: 80
EOF
```

I have defined a `host` value of `hello.test.com`. Obviously this domain name is not resolvable on the internet so we need to adjust our local system to allow this domain to be resolved locally. To do this you will need to edit your local `/etc/hosts` file and add the following line.

```
127.0.0.1   hello.test.com
```
Note, we are using the localhost address which is the same as the listenAddress that we included in the kind cluster config for worker node that is hosting the ingress controller. Hopefully this is all making sense now.

From your terminal (or web browser), you should be able to access the service.

```
curl hello.test.com
```

```
Server address: 10.244.1.5:80
Server name: hello
Date: 10/Feb/2021:16:47:27 +0000
URI: /
Request ID: 50eec66a0f425819257a1190a1cb1ab7
```

CONGRATULATIONS!!!
You have successfully setup and configured a local kubernetes environment. You can now explore, deploy, and configure additional resources in our cluster to your heart's content.
