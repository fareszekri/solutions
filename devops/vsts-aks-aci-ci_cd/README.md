Elastic VSTS Build and Release  on Kubernetes + ACI
 =================================
My work is based on  Anthony Chu @anthonych Blog post [Running a VSTS Docker Agent in Kubernetes](https://anthonychu.ca/post/vsts-agent-docker-kubernetes/). My intent is to extend the build Agents to ACI whenever we face an increasing load on the diffrent nodes. Beyond this section I would assume that you've went through his blog post and already tried the diffrent steps but I'll include a summary of the steps for reference.

Prerequisites
------------
1. Windows Subsystem for Linux (Recommended) [Installation instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
2. Azure CLI - [Installation instructions](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
3. A running Kubernetes Cluster - [AKS Instructions](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster)
4. Helm - [Installation Instructions](https://github.com/kubernetes/helm/blob/master/docs/install.md)
5. Draft - [Installation Instructions](https://github.com/Azure/draft/blob/master/docs/install.md)
6. kubectl - [Installation Instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
7. Docker - [Installation Instructions](https://docs.docker.com/install/)
8. Active VSTS account - [Create one here if you don't have one](https://go.microsoft.com/fwlink/?LinkId=307137&clcid=0x409&wt.mc_id=o~msft~vscom~getstarted-hero~12778&campaign=o~msft~vscom~getstarted-hero~12778)
9. Visual Studio Code - Optional but you'll love it if you haven't already [Free download here](https://code.visualstudio.com/?wt.mc_id=vscom_freedevoffers)

Steps summary for running a vsts agent on Kubernetes
----------------------------
First things first, let's make sure your environment is properly set. I will be running all demos on a Windows 10 with Windows Subsystem for Linux installed. For any other environement please refer to teh Installation Instructions:
 - Install Azure CLI [![asciicast](https://asciinema.org/a/YKkMrqT2udmVOpoMA8rX0QTvg.png)](https://asciinema.org/a/YKkMrqT2udmVOpoMA8rX0QTvg) 
 - Login to you subscription `az login`

 

We need to setup an AKS cluster, create ACR and configure ACR authentication for AKS, add the VSTS Agent to the Azure Container Regsitry (ACR) then define a Build queue where these agents will be attached.
1. Create AKS : ` az aks create -n fzKb8s -g OssDemos --node-count 1 --generate-ssh-keys`
2. Connect to AKS : `az aks get-credentials -n fzKb8s -g OssDemos`
3. Download and install kubectl : `az aks install-cli`
4. Check kubectl is running and cluster is up : `kubectl get nodes`
5. Create an Azure Container Registry (ACR) : `az acr create -n ossDemosContainerRegistry -g OssDemos --sku Basic -admin-enable true -l westeurope`
6. Configure ACR authentication in order to allow aks to pull containers from ACR  :
```bash
CLIENT_ID=$(az aks show -g OssDemos -n fzKb8s --query "servicePrincipalProfile.clientId" --output tsv)
ACR_ID=$(az acr show -n ossDemosContainerRegistry -g OssDemos --query "id" --output tsv)
az role assignment create --assignee $CLIENT_ID --role Reader --scope $ACR_ID
``` 
## Switching to the VSTS part :
You can work on your own projects if you want, but 
configure virtual kubelet
Create an HPA

create secret in AKS kubectl create secret generic vsts --from-literal=VSTS_TOKEN=onqkk4slpsg4dzxcffey5f5cn6mestx2wxip3lyr7jc35so5xdnq --from-literal=VSTS_ACCOUNT=fareszekri


https://github.com/virtual-kubelet/virtual-kubelet/blob/master/providers/azure/README.md
```bash
az account list -o table
export AZURE_SUBSCRIPTION_ID="2a809161-f58d-4156-b039-dcd9e43d9c84"

helm init

export ACI_REGION=westeurope
az group create --name aci-builders --location "$ACI_REGION"
export AZURE_RG=aci-builders
az ad sp create-for-rbac --name virtual-kubelet -o table
export AZURE_TENANT_ID=<Tenant>
export AZURE_CLIENT_ID=<AppId>
export AZURE_CLIENT_SECRET=<Password>
export VK_RELEASE=virtual-kubelet-for-aks-0.1.3


RELEASE_NAME=virtual-kubelet
NODE_NAME=virtual-kubelet
CHART_URL=https://github.com/virtual-kubelet/virtual-kubelet/raw/master/charts/$VK_RELEASE.tgz

curl https://raw.githubusercontent.com/virtual-kubelet/virtual-kubelet/master/scripts/createCertAndKey.sh > createCertAndKey.sh
chmod +x createCertAndKey.sh
. ./createCertAndKey.sh

helm install "$CHART_URL" --name "$RELEASE_NAME" \
    --set env.azureClientId="$AZURE_CLIENT_ID",env.azureClientKey="$AZURE_CLIENT_SECRET",env.azureTenantId="$AZURE_TENANT_ID",env.azureSubscriptionId="$AZURE_SUBSCRIPTION_ID",env.aciResourceGroup="$AZURE_RG",env.nodeName="$NODE_NAME",env.nodeOsType=Linux,env.apiserverCert=$cert,env.apiserverKey=$key
```

