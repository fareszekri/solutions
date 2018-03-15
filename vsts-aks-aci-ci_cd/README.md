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

 ![login to your subscription](images/azLogin.gif)

We need to setup an AKS cluster, add the VSTS Agent to the Azure Container Regsitry (ACR) then define a Build queue where these agents will be attached.

1. 

