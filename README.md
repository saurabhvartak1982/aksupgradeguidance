# Azure Kubernetes Service Upgrade Guidance
The article discusses the planning for Azure Kubernetes Service related upgrades. <br /><br />
The first 5 sections provide a quick primer as to what are the different concepts involved as far as Azure Kubernetes Service related upgrades are concerned.<br />
Section 6 tries to stitch together all these concepts and provides a point-of-view as to how the AKS Cluster upgrades can be planned and executed. <br /><br />

## 1. Kubernetes cluster logical diagram
**Control Plane:** Core Kubernetes Services <br />
**Data Plane (Nodes in the NodePool):** Nodes running the application workloads <br />
![Kubernetes cluster logical diagram](/images/k8sclusterarchitecturediag.png) <br />
Diagram courtesy - https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#kubernetes-cluster-architecture <br />

Detailed overview on the Control Plane can be found here - https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#control-plane <br />
Detailed overview on the Data Plane (Nodes and NodePools) can be found here - https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#nodes-and-node-pools <br /><br />

## 2. Types of AKS related upgrades
### a. AKS version upgrades for Control Plane and Data Plane
This upgrade type is related to the AKS versions in Control Plane and NodePools. <br />
</b> Control Plane upgrade includes the System Pods running on Nodes as well.<br /><br />
Considering the semanting versioning covention of **Major (X).Minor(Y).Patch(Z)**, new minor version is available almost every 3 - 4 months. Patch versions usually come in the frequency of 1 month or a few times within a month. <br />
Minor versions in the support window are the latest and the previous 2 versions -- **Y, Y-1, Y-2**. <br />
Each Minor version in the support window include 2 of the latest stable patch versions -- **Z, Z-1**. <br /><br />
For the detailed information, please refer to the **Kubernetes version support policy** section at - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#kubernetes-version-support-policy <br /><br />

### b. Node Image Upgrades for the Data Plane Nodes
This upgrade type is related to the Node Images in the AKS NodePools. New OS image is available almost every 1 week for the Linux OS and almost every 1 month for the Windows OS. More information on the Node Image Upgrades here - https://learn.microsoft.com/en-us/azure/aks/node-image-upgrade <br /><br />

### c. Node OS security and kernel updates for the Data Plane Nodes (Linux Nodes)
This upgrade type is related to the OS security fixes or kernel updates for the Nodes in the AKS NodePool. Some of these updates require a node reboot to complete the process. AKS doesn't automatically reboot these Linux nodes to complete the update process. Open-source solutions like KURED to manage the auto-reboot of a particular Node. More information on KURED here - https://github.com/kubereboot/kured <br />
More information on the Node OS security and kernel updates here - https://learn.microsoft.com/en-us/azure/aks/node-updates-kured <br /> <br />

## 3. Ensuring application availability during upgrades
### a. Kubernetes/AKS-based measures
**1. Node Surge** <br />
Node Surge setting indicates the buffer/extra Nodes that are created during the upgrades - at the NodePool level. These extra Node(s) help in minimizing disruptions during the upgrades as well as to tune the upgrade speed. <br />
Please refer to the section **Customize node surge upgrade** for guidance on this configuration here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#customize-node-surge-upgrade <br /><br />

**2. Pod Disruption Budget (PDB)** <br />
Pod Disruption Budget allows a control over how many pods can be down at the same time during voluntary disruptions - like an upgrade process. <br />
More information on Pod Disruption Budgets can be found here - https://kubernetes.io/docs/tasks/run-application/configure-pdb/ <br /><br />

**3. NodePool-level Blue-Green set-up** <br />
What is the primary driver behind NodePool-level Blue-Green set-up? Is it faster or does it better ensure application availability? Or does it help for a better rollback? <br /><br />
Below is the sequence which is to be followed:<br /><br />
a. Upgrade the Control Plane.<br />
b. Create a new NodePool (Green) with the upgraded Kubernetes version.<br />
c. Cordon and drain the older NodePool (Blue). This action will move the workloads to the new NodePool (green).<br />
d. Delete the older NodePool (Blue).<br />

### b. Architecture patterns-based measures
**1. AKS Cluster-level Blue-Green set-up** <br />
AKS Cluster-level Blue-Green set-up is the safest option to perform AKS upgrades. This option gives a better ability to validate the functioning of the applications post an upgrade and it also provides an easy way to perform the roll-back. This option is may be expensive. <br />
Below is the sequence which is to be followed **(one of the many ways)**:<br /><br />
a. Have an AKS Cluster-level Blue-Green set-up in-place. Essentially, it is 2 AKS Clusters (one Blue and one Green) both sitting behind a single L7 load balancer like an Azure Application Gateway or an Azure Front Door. I am assuming there is an active-active set-up of AKS Clusters such that both the clusters service the requests.<br />
b. Turn off the traffic flowing to the Green AKS Cluster. <br />
c. Upgrade the Green AKS Cluster to the higher version of Kubernetes.<br />
d. Sanity test the Green AKS Cluster.<br />
e. Allow the traffic to flow to the Green AKS Cluster.<br />
f. Turn off the traffic flowing to the Blue AKS Cluster.<br />
g. Upgrade the Blue AKS Cluster to the higher version of Kubernetes.<br />
h. Sanity test the Blue AKS Cluster.<br />
i. Allow the traffic to flow to the Blue AKS Cluster.<br /><br />

If an active-active set-up of AKS Clusters is not desired and only one AKS Cluster is to be serving the requests, then the Blue cluster can either be deleted OR can be upgraded and scaled down so that it can be re-purposed as a Green AKS Cluster for the next upgrade.<br /><br />

A reference article on the Blue-Green deployment of AKS Clusters can be found here - https://learn.microsoft.com/en-us/azure/architecture/guide/aks/blue-green-deployment-for-aks <br /><br />

## 4. Available AKS upgrades related information
### AKS Kubernetes Release calendar
AKS Kubernetes Release calendar is used to view the upcoming version releases. The same can be found here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-kubernetes-release-calendar <br />

### AKS Upgrade GitHub release page
Announcements related to the planned date of a new version release and deprecation of the old version. The same can be found here - https://github.com/Azure/AKS/releases <br />
One can also subscribe to the feeds - https://github.com/Azure/AKS/releases.atom <br />

### AKS Release Tracker
Region-wise AKS Release and AKS Node Images status - https://releases.aks.azure.com/ . How to map a particular AKS version to the AKS release --? <br />

### Service Health Notification --?
Notification on planned AKS version removal dates - https://learn.microsoft.com/en-us/azure/service-health/service-health-overview <br />

### Azure Advisor Alerts --?
Notification in case if any of the AKS clusters is out of support - https://learn.microsoft.com/en-us/azure/advisor/advisor-overview . <br />

### az-cli
a. az aks get-upgrades - for fetching the available upgrades for a particular AKS cluster. More info here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-upgrades <br />
b. az aks nodepool get-upgrades - for fetching the available NodeImage upgrades for a particular NodePool. More info here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-get-upgrades <br />

### Microsoft Defender Notifications --? ... need to detail what info is provided by this document? <br />
### Availability of a patch upgrade ---? ... need to detail what info is provided by this document? <br />


## 5. Ways to upgrade an AKS cluster
### a. Manual
Cluster and Node Image upgrade
<b>1. Check for available versions</b> <br />
az aks get-upgrades ---- for fetching the available upgrades  <br />
#####      az aks nodepool get-upgrades ----- for fetching the available node image upgrades only, NOT for AKS upgrades <br />
####     Upgrade the Control Plane <br />
####     Upgrade the Nodepools - 1 NodePool upgrade at a time <br />
###      View upgrade events <br />
###      Verification of the upgrade which has been performed <br />

### b. Automated <br />
#### a. Auto-upgrade channel + Maintenance Window <br />
#####     Type of Auto-upgrades <br />
######        Cluster auto-upgrades <br />
#######         <more detailing here> <br />
######        Node OS auto-upgrades <br />
#######   	<more detailing here> <br />
#####     Maintenance Window <br />
######      Window for Cluster Upgrade <br />
#######	      <more detailing here> <br />
######      Window for Node Upgrade <br />
#######       <more detailing here> <br />

#### b. GitHub Actions OR Cron Jobs <br /><br />

## 6. Day-2 operations related to AKS
Day-2 operations comprise of monitoring and maintenance operations of the AKS cluster. AKS related upgrades are part of the maintenance operations. <br />
Basis the first 5 sections of the document, one of the PoV for the AKS Day-2 operations can be as defined in the below sections:  

## 7. PoV on the Day-2 set-up for AKS cluster upgrades 
###     Cluster-wise and environment-wise planning of AKS upgrades 
####      Cluster-wise planning of upgrades of each environment basis the AKS Release calendar- refer AKS_Cluster_Upgrade_Plan.xlsx 
#####       Frequency of upgrades <br />
######            Need to prescribe the upgrade frequency <br />
#####       Selection of the type of upgrade (automated/manual, upgrade channel, etc. ) basis the environment <br />
###     Check for AKS cluster-related pre-requisites for every AKS cluster
####      Ensure PDBs and Node Surge settings are in place. For Zone-redundant Nodepools ensure that Node Surge is in the multiples of 3 <br />
####      Ensure that the resource quota for the VMs of the NodePool is enough to cater to the NodeSurge
####      Ensure that the IP address availability is enough to cater to the Node Surge -- Kubectl get nnc -- <br />
####      If AKS cluster-level Blue-Green set-up is in place, ensure that the above mentioned pre-requsites are in-place for both the clusters <br />
###     Check for breaking changes 
####      Check for any breaking changes which may result as a result of the upgrade. This can be done using one of the below 4 methods: <br />
#####       a. A quick initial check for any upgrade resulting in changes to the minor version by going through the document here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-components-breaking-changes-by-version 
#####       b. Azure Advisor alerts informing about potential issues due to deprecated APIs - https://learn.microsoft.com/en-us/azure/advisor/advisor-overview 
#####       c. <b>Kubernetes API Deprecations</b> view in the <b>Diagnostic Setting --> Create, Upgrade, Delete and Scale <b /> of the AKS cluster page
#####       d. Raising an advisory support case with Microsoft
###     Sequential upgrades of the AKS clusters different environments basis the schedule defined in AKS_Cluster_Upgrade_Plan.xlsx <br />
###     Post-upgrade alert <br />
###     Post-upgrade update of the AKS_Cluster_Upgrade_Plan.xlsx <br />
###     Automated/Manual Sanity/Perf Testing post an upgrade -  basis the environment <br />

## LTS option




			
			

