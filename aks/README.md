# AKS on Azure China Best Practices
Azure Kubernetes Service is in **Private Preview**, this page gives best practices about how to use AKS on Azure China cloud.
 - Contact AKS China Team: [akscn@microsoft.com](mailto:akscn@microsoft.com)

## 1. How to create AKS on Azure China
Currently AKS on Azure China could only be created by [azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) and only supports `chinaeast2` region
 - How to use [azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) on Azure China
```sh
az cloud set --name AzureChinaCloud
az login
az account list
az account set -s <subscription-name>
```

 - Example: create a `v1.10.8` AKS cluster on `chinaeast2`
```sh
RESOURCE_GROUP_NAME=demo-aks1108
CLUSTER_NAME=demo-aks1108
LOCATION=chinaeast2

# create a resource group
az group create -n $RESOURCE_GROUP_NAME -l $LOCATION

# create AKS cluster with 1 agent node
az aks create -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME --node-count 1 --node-vm-size Standard_DS3_v2 --generate-ssh-keys --kubernetes-version 1.10.8

# wait about 15 min for `az aks create` running complete

# get the credentials for the cluster
az aks get-credentials -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME

# get all agent nodes
kubectl get nodes

# open the Kubernetes dashboard
az aks browse --resource-group $RESOURCE_GROUP_NAME -n $CLUSTER_NAME

# scale up/down AKS cluster nodes 
az aks scale -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME --node-count=2

# delete AKS cluster node
az aks delete -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME

```
 > Get more detailed [AKS set up steps](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough)
 
 > Detailed "az aks" command line manual could be found [here](https://docs.microsoft.com/en-us/cli/azure/aks)

 -  All available kubernetes version on `chinaeast2`
```
az aks get-versions -l chinaeast2 -o table
KubernetesVersion    Upgrades
-------------------  ----------------------
1.11.3               None available
1.11.2               1.11.3
1.10.8               1.11.2, 1.11.3
1.10.7               1.10.8, 1.11.2, 1.11.3
1.9.11               1.10.7, 1.10.8
1.9.10               1.9.11, 1.10.7, 1.10.8
1.8.15               1.9.10, 1.9.11
1.8.14               1.8.15, 1.9.10, 1.9.11
1.7.16               1.8.14, 1.8.15
1.7.15               1.7.16, 1.8.14, 1.8.15
```

## 2. Container registry proxies
Since some container registries like `gcr.io`, `docker.io` are not accessible or very slow in China, we have set up container registry proxies in `chinaeast2` region now:

| global | registry proxy in Azure China | Example |
| ---- | ---- | ---- |
| dockerhub (docker.io) | [dockerhub.azk8s.cn](http://mirror.azk8s.cn/help/docker-registry-proxy-cache.html) | dockerhub.azk8s.cn/library/centos |
| gcr.io | [gcr.azk8s.cn](http://mirror.azk8s.cn/help/gcr-proxy-cache.html) | gcr.azk8s.cn/google_containers/hyperkube-amd64:v1.9.2 |
| quay.io | [quay.azk8s.cn](http://mirror.azk8s.cn/help/quay-proxy-cache.html) | quay.azk8s.cn/deis/go-dev:v1.10.0 |

> Note:
`k8s.gcr.io` would redirect to `gcr.io/google-containers`, following image urls are identical:
```
k8s.gcr.io/pause-amd64:3.1
gcr.io/google_containers/pause-amd64:3.1
```

## 3. Install kubectl
Original `az aks install-cli` does not work in azure china, follow detailed steps [here](https://mirror.azk8s.cn/help/kubernetes.html)

## 4. Install helm
follow detailed steps [here](https://mirror.azk8s.cn/help/kubernetes.html)

> Note:
All kubernetes related binaries on github could be found under [https://mirror.azk8s.cn/kubernetes](https://mirror.azk8s.cn/kubernetes), e.g. helm, charts, etc.

## 5. Run a demo on AKS cluster
Follow https://github.com/andyzhangx/k8s-demo/tree/master/nginx-server#nginx-server-demo

### Limitations of current AKS Private Preview on Azure China
 - only `chinaeast2` region is supported
 - AKS set up wizard is not available on azure portal, only azure cli command line is supported
 - AKS monitoring and logging are not available, there will be error when clicking on `Monitor containers` and `View logs` links in AKS overview page

### Links
 - Click for trial: [http://aka.ms/aks/chinapreview](http://aka.ms/aks/chinapreview)
  > please make sure you already have an **Azure China** Subscription
 - AKS doc: [https://docs.microsoft.com/en-us/azure/aks/](https://docs.microsoft.com/en-us/azure/aks/) 
  > Chinese version: [https://docs.microsoft.com/zh-cn/azure/aks/](https://docs.microsoft.com/zh-cn/azure/aks/) 
 - [Deploy an Azure Container Service (AKS) cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough)
 - [Frequently asked questions about Azure Container Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/faq#are-security-updates-applied-to-aks-agent-nodes)
