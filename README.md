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
This script will create a three node cluster. 
