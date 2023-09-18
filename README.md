# Azure Kubernetes Service Upgrade Guidance
The article discusses the planning for Azure Kubernetes Service related upgrades. <br /><br />
The **first 5 sections** provide a quick primer as to what are the different concepts involved as far as Azure Kubernetes Service related upgrades are concerned.<br />
**Section 6** and **section 7** tries to stitch together all these concepts and provides a point-of-view as to how the AKS Cluster upgrades can be planned and executed. <br />
**Section 8** introduces an additional option of the Long Term Support (LTS) - not yet available <br /><br />
To prevent duplication, the article relies on the official AKS or Kubernetes documentation and mentions the references to the same, wherever applicable. <br /><br />
## 1. Kubernetes cluster logical diagram
A Kubernetes cluster is logically divided into 2 parts: <br /><br />
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
**1. AKS version upgrade for the Control Plane and the Data Plane** <br /><br /> 
<b>a. Check for available versions</b> <br /><br />
`az aks get-upgrades --resource-group <ResourceGroupName> --name <AKSClusterName> --output table` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-upgrade  <br /><br />

<b>b. Check for the Kubernetes versions of the Nodes in the NodePools</b> <br /><br />
`az aks nodepool list --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --query "[].{Name:name,k8version:orchestratorVersion}" --output table` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-list <br /><br />

<b>c. Upgrade the Control Plane</b> <br /><br />
`az aks upgrade --resource-group <ResourceGroupName> --name <AKSClusterName> --control-plane-only --no-wait --kubernetes-version <KubernetesVersion>` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-upgrade  <br /><br /> 

<b>d. Upgrade the Data Plane</b> <br /><br />
`az aks nodepool upgrade --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --name <NodePoolName> --no-wait --kubernetes-version <KubernetesVersion>` <br />
Upgrade one NodePool at a time <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade <br /><br />

<b>e. View the upgrade events</b> <br /><br />
`kubectl get events` <br /><br />

<b>f. Verify the upgrade</b> <br /><br />
`az aks show --resource-group myResourceGroup --name myAKSCluster --output table` <br /><br />
`az aks nodepool list --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --query "[].{Name:name,k8version:orchestratorVersion}" --output table` <br /><br />

**2. Node Image Upgrades for the Data Plane Nodes** <br /><br />
<b>a. Check for the available Node Images for a NodePool</b> <br /><br />
`az aks nodepool get-upgrades --resource-group MyResourceGroup --cluster-name MyManagedCluster --nodepool-name MyNodePool` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-get-upgrades <br /><br />

<b>b. Upgrade the Node Image of the NodePool</b> <br /><br /> 
`az aks nodepool upgrade --cluster-name --name --resource-group --node-image-only` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade <br /><br />

<b>c. Verify the Node Image version of the NodePool </b> <br /><br />
`az aks nodepool show --cluster-name --name --resource-group` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-show <br /><br />

### b. Automated <br />
#### a. Auto-upgrade channel + Planned Maintenance <br />
The **Auto-upgrade** feature enables you to automatically upgrade the Cluster/NodePool of an AKS Cluster. The same is achieved by configuring the desired auto-upgrade channel.<br /><br />
#####     Type of Auto-upgrades <br />
<b>1. Cluster auto-upgrades</b> <br /><br/>
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster <br /><br />
<b>2. Node OS auto-upgrades</b> <br /><br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-node-image <br /><br />
#####     Maintenance Window <br />
The **Planned Maintenance** feature enables you to schedule the auto-upgrades in a cadence of your choice so that the impact can be minimized. The scheduling of the auto-upgrades can be done at the below two levels: <br /><br />
<b>1. Scheduling of the auto-upgrades at the Cluster-level</b> <br />
<b>2. Scheduling of the auto-upgrades at the NodePool-level</b> <br />
Documentation link related to **Planned Maintenance** here - https://learn.microsoft.com/en-us/azure/aks/planned-maintenance <br /><br />

#### b. GitHub Actions OR Cron Jobs <br /><br />
Automatic upgrades to the AKS Clusters can also be automatically scheduled by using GitHub Actions or any other similar scheduling tool. <br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/node-upgrade-github-actions <br /><br />

<b>Very Important - Whichever approach is decided, it is important to ensure that the AKS Cluster is always running in the supported version and on one of the latest Node Images.</b> <br /><br />

## 6. Day-2 operations related to AKS
Day-2 operations comprise of monitoring and maintenance operations of the AKS cluster. AKS related upgrades are part of the maintenance operations. <br />
Basis the foundation mentioned in the sections 1 to 5 of this article, one of the approaches for carrying out the AKS related Day-2 operations can be as defined in the below sections:  

## 7. PoV on the Day-2 set-up for AKS cluster upgrades 
### 1. Cluster-wise and environment-wise planning of AKS Upgrades
Below are the considerations for planning of the upgrades related to the AKS Clusters:<br /><br />
a. The AKS Upgrades should be planned in a way that the clusters in the lower environments are upgraded first and subsequently the ones in the higher environments are upgraded.<br /><br />
b. The upgrade planning should ensure that **ALL** the clusters in every environment are always in the supported versions' window at any point of time. Please refer to the Kubernetes version support policy here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#kubernetes-version-support-policy <br /><br />
c. The upgrade planning should ensure that any application is sanity/perf tested first on the upgraded lower environment before it's corresponding AKS Cluster on the higher environment is upgraded. <br /><br />
d. The type of AKS Upgrade (manual/automated) should be clearly noted for better planning. <br /><br />
e. Sufficient time should be planned for the check on upgrade pre-requisites and a sufficient buffer should be planned for fixing the breaking changes - if any. <br /><br />
f. Documentation links mentioned in section 4 of this article should be always referred for planning. <br /><br />
To facilitate better planning related to AKS Upgrades, a sample upgrade plan can be created basis the reference template attached in the form of **AKS_Cluster_Upgrade_Plan.xlsx** <br /><br />
**Optional screenshot of the xlsx to go here**<br /><br />

####       Frequency of upgrades <br />
The **Control Plane** of the AKS Cluster is to be upgraded with a cadence that it is always in the supported version. Irrespective of the upgrade method used, **at a minimum**, the upgrade plan should be **equivalent** to the **patch** channel defined for cluster auto-upgrade. <br /><br />
Documentation link detailing the cluster auto-upgrade channels here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster#use-cluster-auto-upgrade <br /><br />

The **Data Plane** of the AKS Cluster is to be upgraded with a cadence such that its Node Pools are always running with the latest Node Image available for that particular AKS version. Irrespective of the upgrade method used, **at a minimum**, the upgrade plan should be **equivalent** to the **NodeImage** channel defined for cluster auto-upgrade. <br /><br />
Documentation link detailing the node OS auto-upgrade channels here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-node-image#using-node-os-auto-upgrade <br /><br />

####       Selection of the type of upgrade (automated/manual, upgrade channel, etc. ) basis the environment -- ? <br />
### 2. Check for AKS cluster-related pre-requisites for every AKS cluster
Before the upgrade of any AKS Cluster on any environment, a mechanism of ensuring if the below minimum pre-requisites are met should be in place: <br /><br />
a. Ensure **PDB** and **Node Surge** settings are in place. For Zone-redundant Nodepools ensure that Node Surge is in the multiples of 3 <br /><br />
b. Ensure that the resource quota for the VMs of the NodePool is enough to cater to the **Node Surge**<br /><br />
c. Ensure that the **IP address availability** is enough to cater to the **Node Surge** and **Pods Per Node** configurations -- Kubectl get nnc --? . More information here - https://learn.microsoft.com/en-us/azure/aks/azure-cni-overview#plan-ip-addressing-for-your-cluster <br /><br />
d. If **AKS cluster-level Blue-Green** set-up is in place, ensure that the above mentioned pre-requsites are in-place for both the clusters - **Blue** and **Green** <br /><br />

### 3. Check for breaking changes  for every AKS Cluster
Kubernetes upgrades - especially when the minor version upgrades - may result in breaking changes. The below steps should be carried out to check if there are any breaking changes: <br /><br />
a. A quick initial check for any upgrade resulting in changes to the minor version by going through the document here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-components-breaking-changes-by-version <br /><br />
b. Azure Advisor alerts informing about potential issues due to deprecated APIs - https://learn.microsoft.com/en-us/azure/advisor/advisor-overview -- ? <br /><br />
c. <b>Kubernetes API Deprecations</b> view in the <b>Diagnostic Setting --> Create, Upgrade, Delete and Scale </b> of the AKS cluster page. Documentation link here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#remove-usage-of-deprecated-apis-recommended <br /><br />
d. Raising an advisory support case with Microsoft -- ? <br /><br />

### 4. Upgrades of the AKS clusters in different environments basis the planning done as per section 7.1 of this article
This is the step in which the actual execution of upgrades for the AKS Clusters take place. It is to be ensured that all the steps mentioned in sections 7.1, 7.2 and 7.3 are carried out for that particular cluster which is getting upgraded. <br /><br />

### 5. Post-upgrade alert
For every AKS Cluster, an Azure Monitor alert should be set which informs when an AKS Cluster is upgraded. This is especially important when an automated upgrade method is employed to upgrade an AKS Cluster. <br /><br />

### 6. Post-upgrade update of the AKS_Cluster_Upgrade_Plan.xlsx
Whenever any AKS Cluster is upgraded, the AKS upgrade planner (**AKS_Cluster_Upgrade_Plan.xlsx** in our reference template example) should be updated with all the details. The AKS upgrades across all the environments should be properly tracked. <br /><br />

### 7. Automated/Manual Sanity/Perf Testing post an upgrade -  basis the environment
Every AKS Cluster upgrade should be be followed by Sanity OR Performance tests for all the applications which are hosted on that upgraded AKS Cluster. <br /><br />

## 8. LTS option (not yet available as of this writing)
More information here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#long-term-support-lts <br /><br />



			
			

