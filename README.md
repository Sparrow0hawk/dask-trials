# Simple dask stuff


## Setting up a Dask Kubernetes cluster

A brief walkthrough of how to setup an Azure Kubernetes Service cluster on which we'll use helm to install Dask and Jupyterlab. We'll then be able to connect to the service via the web.

### Requirements

I did this all locally on Linux with the following tools installed:
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- [Helm](https://helm.sh/docs/intro/install/)
- Kubectl (installed via CLI, see below)

### Walkthrough

Some of the steps here are adapted from the [Zero to JupyterHub walkthrough](https://zero-to-jupyterhub.readthedocs.io/en/latest/microsoft/step-zero-azure.html). 

With the requirements installed we can proceed using the command line to connect to our Azure account.

```bash
$ az login
```


```bash

# get a list of available subscriptions
az account list --refresh --output table

# set client to use one specific subscription
az account set --subscription <YOUR-CHOSEN-SUBSCRIPTION-NAME>

# create a resource group (if you don't have one already you want to use)
az group create \
              --name=<RESOURCE-GROUP-NAME> \
              --location=centralus \
              --output table

# create an aks cluster with some standard settings
az aks create \
    --name <CLUSTER-NAME> \
    --resource-group <RESOURCE-GROUP-NAME> \
    --node-count 3 \
    --node-vm-size Standard_D2s_v3 \
    --output table

# install the kubectl
az aks install-cli

# fetch your AKS clusters credentials for kubectl to use to communicate with k8 cluster
az aks get-credentials \
             --name <CLUSTER-NAME> \
             --resource-group <RESOURCE-GROUP-NAME> \
             --output table

# download the dask helm charts
helm repo add dask https://helm.dask.org
helm repo update

# use helm to install the dask chart
# the --set options here are crucial to ensure the cluser is publicly exposed
helm install dask-kube dask/dask \
             --namespace dask-kube \
             --create-namespace \
             --set scheduler.serviceType=LoadBalancer \
             --set jupyter.serviceType=LoadBalancer

```

After running helm install you'll get a large output in the terminal which contains details you can run in the terminal to get the address at which to navigate to connect to the JupyterLab session on your K8 cluster.
