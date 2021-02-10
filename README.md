# Local Kind Kuberentes Environment
It's important to be able to development and test against a Kubneretes cluster which you can setup and manage locally on your laptop. My personally favorite tool is to use [Kind](https://kind.sigs.k8s.io/) to setup a multi-node Kubernetes cluster locally. This repository contains documentation and resources to help one construct a Kind environment that actually works on their laptop including proper ingress support. 

Note, the instructions in this respository assume you are using a Mac for development. Variations may exist if you use Linux or Windows.

# Prerequisites
There are some tools that you will need to setup prior to configuring your Kind environment.

## Docker Desktop
Kind runs Kubernetes nodes as containers; therefore, you must have [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac) installed. 

Following the instructions [here](https://hub.docker.com/editions/community/docker-ce-desktop-mac) to install Docker Desktop for Mac. Nothing difficult, just follow the instructions.

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

# Configure Docker Desktop
You will need to adjust the settings of your Docker Desktop to ensure that the kubernetes clusters work well. 

Go ahead and start Docker Desktop if you haven't already.

From the menu select **Preferences**.

Go to the **Resources** section to adjust the CPU and Memory allocated to containers from your laptop. I would recommend adjusting the values to be at least 4 cores for CPUs and 8GB for Memory. You can go higher if you wish but these values should be sufficient for most local development needs.

![Docker Desktop Preferences](/images/docker-preferences.png)

Click **Apply & Restart** button.

# Create a multi-node cluster
I'm going to walk you through creating a simple three node cluster with a non-HA control node.
Kind uses a simple YAML file to describe the cluster. I'm not going to walk you through every setting but I will explain some key points that you will want to use.

You can kick off the creation now and then read about the key points in the sections that follow.

```
kind create cluster --name xl-cluster --config https://raw.githubusercontent.com/dcberg/kind/main/scripts/xl-cluster.yaml
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

