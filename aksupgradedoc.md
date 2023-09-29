# Upgrade Guidance for Azure Kubernetes Service

The article explains the considerations and best practices related to upgrading your Azure Kubernetes Service clusters. <br /><br />

## Introduction

Upgrading your Azure Kubernetes Service (AKS) cluster is an essential task to ensure that your applications are running on the latest and most secure version of Kubernetes. However, upgrading an AKS cluster can be a complex process that requires careful planning and execution to avoid downtime or data loss. In this guide, we will provide guidance on how to upgrade your AKS cluster to the latest version, including best practices and tips to ensure a smooth and successful upgrade process.

Upgrading an AKS cluster offers several benefits, including:

1. **Security**: Upgrading to the latest version of Kubernetes ensures that your cluster is running on the most secure version of the platform, with the latest security patches and bug fixes.
2. **Performance**: Newer versions of Kubernetes often come with performance improvements, such as faster networking and more efficient resource utilization, which can improve the overall performance of your applications.
3. **Stability**: Upgrading to a newer version of Kubernetes can help improve the stability of your cluster by fixing bugs and addressing issues that may have been present in previous versions.
4. **Feature enhancements**: Newer versions of Kubernetes often come with new features and enhancements that can help improve the functionality and capabilities of your cluster.
5. **Support**: Running your AKS cluster on a supported version of Kubernetes ensures that you can receive support from the Kubernetes community and Microsoft, which can be critical in the event of issues or outages.

**Note**: This guide is opinionated and based on a general variety of customers. There could be specific scenarios where this guide may not be able to help you put all the best practices in place. We recommend you contact Microsoft Specialists and Support for up-to-date and contextual guidance. <br />
This guide is divided into three main sections as below: <br /><br />
..* - **Prepare for Upgrade** - This section will involve guidance around planning for the AKS upgrade. <br />
..* - **Execute the Upgrade** - This section will be the list of things to consider when upgrading. <br />
..* - **Post Upgrade** - This section will be a set of steps to validate a successful upgrade as well as measure impact to your application. <br />
 
## Prepare for Upgrade
Here are the things that you will consider when preparing for an upgrade

1. **Upgrade Schedule and Propagation across Environments** - Cluster-wise and Environment-wise planning of AKS upgrades <br /><br />
a. The AKS upgrades should be planned in a way that the clusters in the lower environments are upgraded first and subsequently the ones in the higher environments are upgraded.<br /><br />
b. The upgrade planning should ensure that **ALL** the clusters in every environment are always in the supported versions' window at any point of time. Please refer to the Kubernetes version support policy here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#kubernetes-version-support-policy <br /><br />
c. The upgrade planning should ensure that any application is sanity/perf tested first on the upgraded lower environment before it's corresponding AKS Cluster on the higher environment is upgraded. <br /><br />
d. The type of AKS upgrade (manual/automated), upgrade frequency and duration of the upgrades should always be considered for better planning. <br /><br />
e. Sufficient time should be planned for the check on upgrade pre-requisites and a sufficient buffer should be planned for fixing the breaking changes - if any. <br /><br />
f. Below are different modes which help in knowing the availability of the upcoming AKS versions which should be always referred for planning: <br />
**f.1. AKS Kubernetes Release calendar** - To view the upcoming version releases - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-kubernetes-release-calendar <br />
**f.2. AKS Upgrade GitHub release page** - Announcements related to the planned date of a new version release and deprecation of the old version - https://github.com/Azure/AKS/releases <br />
**f.3. AKS Release Tracker** - Region-wise AKS Release and AKS Node Images status - https://releases.aks.azure.com/ <br />
**f.4. AKS events with Azure Event Grid** <br />
AKS events can be subscribed to using the Azure Event Grid. One of the Event Type is **Microsoft.ContainerService.NewKubernetesVersionAvailable** which gets triggered when the list of available Kubernetes versions is updated. <br />
Available event types for AKS - https://learn.microsoft.com/en-us/azure/event-grid/event-schema-aks?tabs=event-grid-event-schema <br />
Subscribing to the AKS events with Azure Event Grid - https://learn.microsoft.com/en-us/azure/aks/quickstart-event-grid?tabs=azure-cli <br />
**f.5. az-cli** <br />
**f.5.a. az aks get-upgrades** - for fetching the available upgrades for a particular AKS cluster. <br />
![AKS Cluster Version Get Upgrades](/images/AKSClusterVersionGetUpgrades.png) <br />
More info here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-upgrades <br />
**f.5.b. az aks nodepool get-upgrades** - for fetching the available NodeImage upgrades for a particular NodePool. <br />
![AKS NodePool Get Upgrades](/images/AKSNodeImageGetUpgrades.png) <br />
More info here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-get-upgrades <br /><br />


To facilitate better planning related to AKS upgrades, a sample upgrade plan can be created basis the reference template attached in the form of **AKS_Cluster_Upgrade_Plan.xlsx**. Below is the screenshot of the same: <br /><br />
![AKS Cluster Upgrade Plan](/images/AKS_Cluster_Upgrade_Plan.png) <br />

2. **Upgrade Type**<br />
There are 2 main categories of AKS-related upgrade types - **User Initiated Upgrade** and **Azure Initiated Upgrade**.<br />
**User Intitiated Upgrades** are further classified into the below 3 types:<br />
- **AKS version upgrade for the Control Plane and the Data Plane**: This upgrade type is related to the AKS versions in Control Plane and NodePools
- **Node Image Upgrades for the Data Plane Nodes**: This upgrade type is related to the Node Images in the AKS NodePools. New OS image is available almost every 1 week for the Linux OS and almost every 1 month for the Windows OS. **An updated Node Image contains up-to-date OS security patches, kernel updates, Kubernetes security updates, newer versions of binaries like kubelet, and component version updates.**
- **Node OS security and kernel updates for the Data Plane Nodes (Linux Nodes)**:This upgrade type is related to the OS security fixes or kernel updates for the Nodes in the AKS NodePool. The default value of this option is **None** - which means these patches wont be applied by default.<br /><br />
The **User Intitiated Upgrades** can be carried out using some **manual** steps and also in an **automated** way. Both the approaches are explained as below: <br /><br />
**2.a. Manual upgrade**<br />
**2.a.1. AKS version upgrade for the Control Plane and the Data Plane**<br />
**2.a.2. Node Image Upgrades for the Data Plane Nodes**<br />
**2.b. Automated upgrade**<br />
**2.b.a. Auto-upgrade channel + Planned Maintenance** <br />
The **Auto-upgrade** feature enables you to automatically upgrade the Cluster/NodePool of an AKS Cluster. The same is achieved by configuring the desired auto-upgrade channel.<br /><br />
**Type of Auto-upgrades**<br />
<b>1. Cluster auto-upgrades</b> <br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster <br /><br />
<b>2. Node OS auto-upgrades</b> <br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-node-image <br /><br />
The **Planned Maintenance** feature enables you to schedule the auto-upgrades in a cadence of your choice so that the impact can be minimized. The scheduling of the auto-upgrades can be done at the below two levels: <br />
<b>1. Scheduling of the auto-upgrades at the Cluster-level</b> <br />
<b>2. Scheduling of the auto-upgrades at the NodePool-level</b> <br />
Documentation link related to **Planned Maintenance** here - https://learn.microsoft.com/en-us/azure/aks/planned-maintenance <br /><br />
**2.b.b. GitHub Actions OR Cron Jobs** <br />
Automatic upgrades to the AKS clusters can also be automatically scheduled by using GitHub Actions or any other similar scheduling tool. <br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/node-upgrade-github-actions <br /><br />
Selection of the upgrade type - Automated OR Manual depends primarily on the below 2 factors: <br />
a. Maturity of the DevOps practices <br />
b. Risk appetite <br /><br />
If the DevOps maturity is high and if there is a high risk appetite, then one can go with the automated upgrades for all the environments. Or else a more controlled approach of manual upgrades can be chosen.<br />
A mixed approach can also be considered where the lower environments can be configured for auto-upgrades (to reduce manual effort) and the higher environments can be upgraded manually. <br /><br />
<b>Very Important - Whichever approach is decided, it is important to ensure that the AKS Cluster is always running in the supported version and on one of the latest Node Images.</b> <br /><br />
3. **Upgrade Frequency**<br />
The **Control Plane** of the AKS Cluster is to be upgraded with a cadence such that it is always in the supported version. Irrespective of the upgrade method used (manual/automated), **at a minimum**, the upgrade plan should be **equivalent** to the **patch** channel defined for cluster auto-upgrade. Kindly note that the **patch** channel will not upgrade the **minor version**. So if a particular AKS Cluster configured with the **auto-upgrade channel** of type **patch** is on a particular **minor version** and if that **minor version** goes out of support, then the AKS Cluster will need to be upgraded to the next minor version **manually**.  <br /><br />
Documentation link detailing the cluster auto-upgrade channels here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster#use-cluster-auto-upgrade <br /><br />

The **Data Plane** of the AKS Cluster is to be upgraded with a cadence such that its Node Pools are always running with the latest Node Image available for that particular AKS version. Irrespective of the upgrade method used (manual/automated), **at a minimum**, the upgrade plan should be **equivalent** to the **NodeImage** channel defined for cluster auto-upgrade. <br /><br />
Documentation link detailing the node OS auto-upgrade channels here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-node-image#using-node-os-auto-upgrade <br /><br />
4. **Upgrade Duration**<br />
The upgrade duration of the Data Plane (NodePool) can be reduced by using the below 2 measures:<br /> 
**4.a. Node Surge** <br />
Node Surge setting indicates the buffer/extra Node(s) that are created during the upgrades - at the NodePool level. These extra Node(s) help in minimizing disruptions during the upgrades as well as to tune the upgrade speed. <br />
Please refer to the section **Customize node surge upgrade** for guidance on this configuration here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#customize-node-surge-upgrade <br /><br />
**4.b. NodePool-level Blue-Green set-up** <br />
The primary driver behind having the NodePool-level Blue-Green set-up is to make the NodePool upgrades faster. <br /><br />
Below is the sequence which is to be followed for NodePool-level Blue-Green set-up:<br />
a. Upgrade the Control Plane with the desired Kubernetes version.<br />
b. Create a new NodePool (Green) with the upgraded Kubernetes version.<br />
c. Cordon and drain the older NodePool (Blue). This action will move the workloads to the new NodePool (green).<br />
d. Delete the older NodePool (Blue).<br /><br />
More information on Cordon + Drain of the AKS NodePool can be found here - https://learn.microsoft.com/en-us/azure/aks/resize-node-pool?tabs=azure-cli <br /><br />
**5. Identify deprecated APIs and impact** <br />
Kubernetes upgrades - especially when the minor version upgrades - may result in breaking changes. The below steps should be carried out to check if there are any breaking changes: <br />
a. A quick initial check for any upgrade resulting in changes to the minor version by going through the document here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-components-breaking-changes-by-version <br /><br />
b. <b>Kubernetes API Deprecations</b> view in the <b>Diagnostic Setting --> Create, Upgrade, Delete and Scale </b> of the AKS cluster page.<br />
![AKS Breaking Changes](/images/AKSBreakingChanges.png) <br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#remove-usage-of-deprecated-apis-recommended <br /><br />

## Execute the Upgrade
When you are ready for executing the upgrades - please ensure the following things. 

### Pre-Upgrade Validation 
- IP addresses availability - Azure CNI
Ensure that the **IP address availability** is enough to cater to the **Node Surge** and **Pods Per Node** configurations. More information here - https://learn.microsoft.com/en-us/azure/aks/azure-cni-overview#plan-ip-addressing-for-your-cluster <br /><br />
If the AKS Cluster is making use of **Overlay OR Dynamic-IP-Allocation** mode, then one can make use of **Node Network Configuration (NNC)** by using the **kubectl get nnc** command to know the number of Pod IPs allocated per node. Example as below: <br />
![kubectl get nnc](/images/nnc.png) <br /><br />
More information on Azure CNI Overlay networking in AKS here - https://learn.microsoft.com/en-us/azure/aks/azure-cni-overlay <br />
More information on Azure CNI networking for Dynamic IP allocation in AKS here - https://learn.microsoft.com/en-us/azure/aks/configure-azure-cni-dynamic-ip-allocation <br /><br />
Also to monitor **IP subnet usage** using a Workbook, the same can be done by using the Workbook by the name **Subnet IP Usage**. More information here - https://learn.microsoft.com/en-us/azure/aks/configure-azure-cni-dynamic-ip-allocation#monitor-ip-subnet-usage <br /><br />
- Resource quota availability
Ensure that the resource quota for the VMs of the NodePool is enough to cater to the configured value of **Node Surge**<br /><br />
- Pod Resiliency and placement
Pod Disruption Budget allows a control over how many pods can be down at the same time during voluntary disruptions - like an upgrade process.
More information on Pod Disruption Budgets can be found here - https://kubernetes.io/docs/tasks/run-application/configure-pdb/ <br /><br />
- Stateful Sets --**??**
For Zone-redundant Nodepools ensure that Node Surge is in the multiples of 3 <br /><br />
- Downtime
Kubernetes/AKS Upgrades can be disruptive. To minimize the disruptions, certain measures can be put in place as below: <br />
**1. Node Surge** <br />
**2. Pod Disruption Budget (PDB)** <br />
**3. NodePool-level Blue-Green set-up**<br />
**4. AKS Cluster-level Blue-Green set-up** <br />
AKS Cluster-level Blue-Green set-up is the safest option to perform the AKS upgrades. This option gives a better ability to validate the functioning of the applications post an upgrade and it also provides an easy way to perform the roll-back. This option is may prove expensive - cost wise. <br /><br />
![AKS Blue-Green set-up](/images/AKSBlueGreen.png) <br />
Diagram courtesy - https://learn.microsoft.com/en-us/azure/architecture/guide/aks/blue-green-deployment-for-aks#architecture<br />

Below is the sequence which is to be followed **(one of the many ways)**:<br /><br />
a. Have an AKS Cluster-level Blue-Green set-up in-place. Essentially, it is 2 AKS Clusters (one Blue and one Green) both sitting behind a single L7 load balancer like an Azure Application Gateway or an Azure Front Door. I am assuming there is an active-active set-up of AKS Clusters such that both the clusters service the requests.<br /><br />
b. Turn off the traffic flowing to the Green AKS Cluster. <br /><br />
c. Upgrade the Green AKS Cluster to the higher version of Kubernetes.<br /><br />
d. Sanity test the Green AKS Cluster.<br /><br />
e. Allow the traffic to flow to the Green AKS Cluster.<br /><br />
f. Turn off the traffic flowing to the Blue AKS Cluster.<br /><br />
g. Upgrade the Blue AKS Cluster to the higher version of Kubernetes.<br /><br />
h. Sanity test the Blue AKS Cluster.<br /><br />
i. Allow the traffic to flow to the Blue AKS Cluster.<br /><br />

If an active-active set-up of AKS Clusters is not desired and only one AKS Cluster is to be serving the requests, then the Blue cluster can either be deleted OR can be upgraded and scaled down so that it can be re-purposed as a Green AKS Cluster for the next upgrade.<br /><br />

A reference article on the Blue-Green deployment of AKS Clusters can be found here - https://learn.microsoft.com/en-us/azure/architecture/guide/aks/blue-green-deployment-for-aks <br /><br />

### Upgrade Steps 

#### Manual

- Downtime 
- Commands
**a. AKS version upgrade for the Control Plane and the Data Plane** - The broader steps involved in the **manual** upgrade are as below: <br />
<b>a. Check for available versions</b> <br />
`az aks get-upgrades --resource-group <ResourceGroupName> --name <AKSClusterName> --output table` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-upgrade <br />
<b>b. Check for the Kubernetes versions of the Nodes in the NodePools</b> <br />
`az aks nodepool list --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --query "[].{Name:name,k8version:orchestratorVersion}" --output table` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-list <br />
<b>c. Upgrade the Control Plane</b> <br />
`az aks upgrade --resource-group <ResourceGroupName> --name <AKSClusterName> --control-plane-only --no-wait --kubernetes-version <KubernetesVersion>` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-upgrade  <br />
<b>d. Upgrade the Data Plane</b> <br />
`az aks nodepool upgrade --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --name <NodePoolName> --no-wait --kubernetes-version <KubernetesVersion>` <br />
Upgrade one NodePool at a time <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade <br />
<b>e. View the upgrade events</b> <br />
`kubectl get events` <br />
**b. Node Image Upgrades for the Data Plane Nodes**. The broader steps involved in the **manual** upgrade are as below: <br />
<b>a. Check for the available Node Images for a NodePool</b> <br />
`az aks nodepool get-upgrades --resource-group MyResourceGroup --cluster-name MyManagedCluster --nodepool-name MyNodePool` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-get-upgrades <br />
<b>b. Upgrade the Node Image of the NodePool</b> <br />
`az aks nodepool upgrade --cluster-name --name --resource-group --node-image-only` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade <br />
<b>c. Verify the Node Image version of the NodePool </b> <br />
`az aks nodepool show --cluster-name --name --resource-group` <br />
Documentation link here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-show <br /><br />


- Monitor
- App changes deployment

#### Automated 

- Downtime
- Monitor
- App changes deployment

## Post Upgrade
When you are ready for executing the upgrades - please ensure the following things. 

### Checks
- Version Checks
<b>Verify the upgrade</b> <br />
`az aks show --resource-group myResourceGroup --name myAKSCluster --output table` <br />
`az aks nodepool list --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --query "[].{Name:name,k8version:orchestratorVersion}" --output table` <br />
Reference documentation here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli <br /><br />  
- Application Health Checks

### Troubleshoot Upgrade Failures
Please refer to the links in the section **Troubleshoot upgrade operations** in the document - https://learn.microsoft.com/en-us/troubleshoot/azure/azure-kubernetes/welcome-azure-kubernetes <br />
- Control Plane Upgrade failure
- Node failure
- Application pod failure

### Propogation to next environment

- Maintenance Schedule
- Infra As Code changes 

