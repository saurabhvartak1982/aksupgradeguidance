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

AKS Upgrades are usually a very seamless affair. Due to the resiliency built inside of AKS, and the way AKS upgrade works, one can be assured of minimal impact to their applications during upgrades. However, there can be edge scenarios that may result in issues arising out of upgrades. Hence it is important that when you decide of your upgrade process - you will need to put some thoughts into how you will be executing the upgrades. Some of the decision criteia for performing upgrades are 

### Upgrade Type

An AKS upgrade can happen in two manner - **User Initiated Upgrade** and **Azure Initiated Upgrade**.<br />

#### User Initiated Upgrade
A User Initiated Upgrade is preferred when the owner requires control of the upgrade operations and would want to put checks in place to avoid any unintentional impact due to upgrade. This can be done manually, step by step, or a user manually triggers a script/pipeline that will perform the upgrade. In either case, the user decides on 
####* When to do the upgrade - Deciding on an upgrade window that has minimal impact on the application.  
####* What steps to take before the upgrade - This could be scaling up the cluster and the application relicas to avoid any potential impact, or to turn off any alerts that get triggered due to application replica downtime, etc. 
####* What steps to take after the upgrade - This could be performing regression testing, or scaling the cluster down, or turning on the alerts that were turned off, etc. 

Azure provides the following tools for customers to perform User Initiated Upgrade
##### AKS Kubernetes Release calendar - To view the upcoming version releases - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-kubernetes-release-calendar <br />
##### AKS Upgrade GitHub release page - Announcements related to the planned date of a new version release and deprecation of the old version - https://github.com/Azure/AKS/releases <br />
##### AKS Release Tracker - Region-wise AKS Release and AKS Node Images status - https://releases.aks.azure.com/ <br />
##### AKS events with Azure Event Grid <br />
AKS events can be subscribed to using the Azure Event Grid. One of the Event Type is **Microsoft.ContainerService.NewKubernetesVersionAvailable** which gets triggered when the list of available Kubernetes versions is updated. <br />
Available event types for AKS - https://learn.microsoft.com/en-us/azure/event-grid/event-schema-aks?tabs=event-grid-event-schema <br />
Subscribing to the AKS events with Azure Event Grid - https://learn.microsoft.com/en-us/azure/aks/quickstart-event-grid?tabs=azure-cli <br />
##### az-cli <br />
##### az aks get-upgrades - for fetching the available upgrades for a particular AKS cluster. <br />
![AKS Cluster Version Get Upgrades](/images/AKSClusterVersionGetUpgrades.png) <br />
More info here - https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-upgrades <br />
##### az aks nodepool get-upgrades - for fetching the available NodeImage upgrades for a particular NodePool. <br />
![AKS NodePool Get Upgrades](/images/AKSNodeImageGetUpgrades.png) <br />
More info here - https://learn.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-get-upgrades <br /><br />

TIP: When doing manual upgrades - you can also upgrade only the control plane first using the """az aks upgrade --control-plane-only""" command, followed by upgrading each individual node-pool seperately. Doing this can ensure that you perform the upgrade process in a small, step by step and calculated manner. 

One can also prepare a pipeline that can be triggered manually or automatically to perform AKS Upgrades. An example of a GitHub Action that can be used to process OS Updates for AKS is provided [https://learn.microsoft.com/en-us/azure/aks/node-upgrade-github-actions here]. The article shows  how one can automate the update process of AKS nodes using GitHub Actions and Azure CLI to create an update task based on cron that runs automatically.  

#### Azure Initiated Upgrade

AKS provides an out of the box automatic upgrade feature, whereby Azure can automatically check for version availability, and then trigger the update based on customer upgrade preferences. By enabling auto-upgrade, you can ensure your clusters are up to date and don’t miss the latest features or patches from AKS and upstream Kubernetes. With properly selected auto-upgrade channels, you can avoid clusters falling into an unsupported version.

To enable automatic upgrades for your AKS cluster, you can use the Azure portal, Azure CLI, or Azure PowerShell. The auto-upgrade feature first upgrades the control plane, and then upgrades agent pools one by one. Stopped node pools will be upgraded during an auto-upgrade operation. The upgrade will apply to nodes when the node pool is started.

It’s important to note that if you’re using cluster auto-upgrade, you can no longer upgrade the control plane only. 

For more details on Auto Upgrades - please refer [https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster Automatic Upgrades in AKS]

####* Auto-upgrade channel + Planned Maintenance
AKS provides a feature called Planned Maintenance that allows you to schedule regular maintenance windows for your AKS cluster. This feature enables you to run both types of maintenance in a cadence of your choice, thereby minimizing any workload impact. 
To use automatic upgrades with planned maintenance, you can specify a maintenance window for your cluster and enable the auto-upgrade feature. The upgrade will start during your specified maintenance window 
For more information on using planned maintenance with AKS auto-upgrades, please refer to Microsoft’s official documentation [https://learn.microsoft.com/en-us/azure/aks/planned-maintenance]

### Upgrade Frequency

AKS follows the upstream kubernetes release cycle. A new minor version of kubernetes is released roughly every 4 months. This version will make its way to AKS roughly after 3-4 months after the upstream release.
Referring to the Semantic Versioning convention of Major (X).Minor(Y).Patch(Z), new minor version in AKS is available almost every 3 - 4 months. Patch versions would usually come in at a higher frequency ~ 1 or more times within a month.

Minor versions in the support window are the latest and the previous 2 versions -- Y, Y-1, Y-2.
Each Minor version in the support window include 2 of the latest stable patch versions -- Z, Z-1.

With Automatic Upgrades - Customer needs to decide on the channel to use - and Azure will upgrade your cluster to the appropriate minor/patch version. For more details on the channels - please refer [https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster#use-cluster-auto-upgrade Channels]

TIP:
The **Control Plane** of the AKS Cluster is to be upgraded with a cadence such that it is always in the supported version. Irrespective of the upgrade method used (manual/automated), **at a minimum**, the upgrade plan should be **equivalent** to the **patch** channel defined for cluster auto-upgrade. Kindly note that the **patch** channel will not upgrade the **minor version**. So if a particular AKS Cluster configured with the **auto-upgrade channel** of type **patch** is on a particular **minor version** and if that **minor version** goes out of support, then the AKS Cluster will need to be upgraded to the next minor version **manually**.  <br /><br />

The **Data Plane** of the AKS Cluster is to be upgraded with a cadence such that its Node Pools are always running with the latest Node Image available for that particular AKS version. Irrespective of the upgrade method used (manual/automated), **at a minimum**, the upgrade plan should be **equivalent** to the **NodeImage** channel defined for cluster auto-upgrade. <br /><br />
Documentation link detailing the node OS auto-upgrade channels here - https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-node-image#using-node-os-auto-upgrade <br /><br />

### Upgrade Duration
An AKS Upgrade process always uses the cordon and drain process to ensure minimum disruptions to your applications during an upgrade. 
The **cordon command** marks a node as unschedulable, which means that no new pods will be scheduled on that node. However, existing pods will continue to run on the node until they are terminated 1.

The **drain command** is used to gracefully terminate all running pods on a node before the node is taken offline for maintenance or upgrade. The drain command ensures that all running pods are rescheduled onto other nodes in the cluster before the node is taken offline.
Due to this process, the duration of an AKS cluster upgrade depends on several factors, including the size of the cluster, the number of nodes, the number of applications running on the cluster, and the complexity of the applications. Lets understand how to configure each of them appropriately

#### Node Surge 
Node Surge setting indicates the buffer/extra Node(s) that are created during the upgrades - at the NodePool level so that more nodes can be marked for upgrades. These extra Node(s) help in minimizing disruptions during the upgrades as well as to tune the upgrade speed. <br />
Please refer to the section **Customize node surge upgrade** for guidance on this configuration here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#customize-node-surge-upgrade <br />

#### Scheduling Contraints
Things such as Node Selectors, Taints and Tolerations, Pod Disruption Budgets will elongate the process of node cordon and draining before performing an upgrade. 

#### NodePool-level Blue-Green set-up
The primary driver behind having the NodePool-level Blue-Green set-up is to make the NodePool upgrades faster. <br /><br />
Below is the sequence which is to be followed for NodePool-level Blue-Green set-up:<br />
a. Upgrade the Control Plane with the desired Kubernetes version.<br />
b. Create a new NodePool (Green) with the upgraded Kubernetes version.<br />
c. Cordon and drain the older NodePool (Blue). This action will move the workloads to the new NodePool (green).<br />
d. Delete the older NodePool (Blue).<br /><br />
More information on Cordon + Drain of the AKS NodePool can be found here - https://learn.microsoft.com/en-us/azure/aks/resize-node-pool?tabs=azure-cli <br /><br />

### Propogation Across Environments
a. The AKS upgrades should be planned in a way that the clusters in the lower environments are upgraded first and subsequently the ones in the higher environments are upgraded.<br /><br />
b. The upgrade planning should ensure that **ALL** the clusters in every environment are always in the supported versions' window at any point of time. Please refer to the Kubernetes version support policy here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#kubernetes-version-support-policy <br /><br />
c. The upgrade planning should ensure that any application is sanity/perf tested first on the upgraded lower environment before it's corresponding AKS Cluster on the higher environment is upgraded. <br /><br />
d. The type of AKS upgrade (manual/automated), upgrade frequency and duration of the upgrades should always be considered for better planning. <br /><br />
e. Sufficient time should be planned for the check on upgrade pre-requisites and a sufficient buffer should be planned for fixing the breaking changes - if any. <br /><br />

To facilitate better planning related to AKS upgrades, a sample upgrade plan can be created basis the reference template attached in the form of **AKS_Cluster_Upgrade_Plan.xlsx**. Below is the screenshot of the same: <br /><br />
![AKS Cluster Upgrade Plan](/images/AKS_Cluster_Upgrade_Plan.png) <br />


**TODO - REVIEW 5. Identify deprecated APIs and impact** <br />
Kubernetes upgrades - especially when the minor version upgrades - may result in breaking changes. The below steps should be carried out to check if there are any breaking changes: <br />
a. A quick initial check for any upgrade resulting in changes to the minor version by going through the document here - https://learn.microsoft.com/en-us/azure/aks/supported-kubernetes-versions?tabs=azure-cli#aks-components-breaking-changes-by-version <br /><br />
b. <b>Kubernetes API Deprecations</b> view in the <b>Diagnostic Setting --> Create, Upgrade, Delete and Scale </b> of the AKS cluster page.<br />
![AKS Breaking Changes](/images/AKSBreakingChanges.png) <br />
Documentation link here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#remove-usage-of-deprecated-apis-recommended <br /><br />

## Execute the Upgrade
Once the Planning is done - and you are now ready to exucute the upgrade (manually or automatic) - here are some best practices and guidance for executing an AKS upgrade.  

### Pre-Upgrade Validation 

#### Underlying Azure Resources

##### Networking
As we have seen during the planning process - an upgrade process will end up creating new nodes (or nodepools). One will need to ensure that the **IP address availability** in your Azure subnet is enough to cater to the **Node Surge** and **Pods Per Node** configurations, depending on the CNI that you have used. For example, in case of Azure CNI - how to plan for your IP addresses that will also take into account upgrade process is provided here [https://learn.microsoft.com/en-us/azure/aks/azure-cni-overview#plan-ip-addressing-for-your-cluster Plan IP Addressing]

TIP
If the AKS Cluster is making use of **Overlay OR Dynamic-IP-Allocation** mode, then one can make use of **Node Network Configuration (NNC)** by using the **kubectl get nnc** command to know the number of Pod IPs allocated per node. Example as below: <br />
![kubectl get nnc](/images/nnc.png) <br /><br />

More information on Azure CNI Overlay networking in AKS here - https://learn.microsoft.com/en-us/azure/aks/azure-cni-overlay <br />
More information on Azure CNI networking for Dynamic IP allocation in AKS here - https://learn.microsoft.com/en-us/azure/aks/configure-azure-cni-dynamic-ip-allocation <br /><br />

TIP: Monitoring IP exhaustion can become difficult. Azure provides a Workbook by the name **Subnet IP Usage**, which can help in monitoring IP Subnet Address Usage.  More information here - https://learn.microsoft.com/en-us/azure/aks/configure-azure-cni-dynamic-ip-allocation#monitor-ip-subnet-usage <br /><br />

##### Resource quota
Ensure that the resource quota for the VMs of the NodePool is enough to cater to the configured value of **Node Surge**<br /><br />

#### Application Related Considerations

##### Pod Resiliency and placement
Pod Disruption Budget allows a control over how many pods can be down at the same time during voluntary disruptions - like an upgrade process.
More information on Pod Disruption Budgets can be found here - https://kubernetes.io/docs/tasks/run-application/configure-pdb/ <br /><br />

TIP: For Zone-redundant Nodepools ensure that Node Surge is in the multiples of 3 <br /><br />

##### Stateful Sets
Applications that have state stored in them, through PVCs have to be slightly careful during upgrades. Stateful sets are used to maintain the state of applications beyond an individual pod lifecycle. Therefore, it’s important to ensure that any data stored in the stateful set is backed up before upgrading the cluster. Backup can be of the following types
a. Workload initiated backups (Recommended) - Depending on the workload that you have, there would be a native way of backing up data of the workload. It is recommended to use native way as that would ensure consistency with the application

b. Azure Kubernetes Service Backup -  is a simple, cloud-native process to back up and restore the containerized applications and data running in AKS clusters. You can configure scheduled backup for cluster state and application data (persistent volumes - CSI driver-based Azure Disks). The solution provides granular control to choose a specific namespace or an entire cluster to back up or restore by storing backups locally in a blob container and as disk snapshots. With AKS backup, you can unlock end-to-end scenarios - operational recovery, cloning developer/test environments, or cluster upgrade scenarios. For more details  https://learn.microsoft.com/en-us/azure/backup/azure-kubernetes-service-backup-overview

#### Additional Considerations

##### Availability Zones
In case you have your AKS clusters that are spread across availability zones - there are some things to keep in mind. We have them documented here. https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#special-considerations-for-node-pools-that-span-multiple-availability-zones 

## Post Upgrade
After an AKS upgrade has completed, it’s important to perform a series of checks to ensure that your workloads are running as expected and that there are no issues with the upgraded cluster 1. Here is a checklist of things to check after an AKS upgrade:

Verify that all nodes in the cluster are running the expected version of Kubernetes.
Check that all pods are running as expected and that there are no issues with any of the workloads.
Verify that all services are running as expected and that there are no issues with any of the services.
Check that all ingress controllers are running as expected and that there are no issues with any of the ingress controllers.
Verify that all persistent volumes (PVs) and persistent volume claims (PVCs) are working as expected.
Check that all custom resource definitions (CRDs) are working as expected.
Verify that all Kubernetes add-ons (such as metrics-server, dashboard, etc.) are working as expected.
Check that all Kubernetes API objects (such as deployments, statefulsets, etc.) are working as expected.
It’s important to note that this is not an exhaustive list, and you may need to perform additional checks depending on your specific use case 1.

### Checks
- Version Checks
<b>Verify the upgrade</b> <br />
`az aks show --resource-group myResourceGroup --name myAKSCluster --output table` <br />
`az aks nodepool list --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> --query "[].{Name:name,k8version:orchestratorVersion}" --output table` <br />
Reference documentation here - https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli <br /><br />  
- Application Health Checks

### Monitor the Upgrade

#### Upgrade Events
AKS provides a detailed set of events that you can use to monitor the progress of the AKS upgrade. These are documented here [https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#view-the-upgrade-events Upgrade Events]

#### Stopped Upgrade Operations
AKS automatically stops upgrade operations consisting of a minor version change if deprecated APIs are detected an error is generated. After receiving the error message, you have two options to mitigate the issue. You can either remove usage of deprecated APIs (recommended) or bypass validation to ignore API changes. To Bypass validation to ignore API breaking changes, use the az aks update command, specifying enable-force-upgrade, and setting the upgrade-override-until property to define the end of the window during which validation is bypassed. If no value is set, it defaults the window to three days from the current time. 

Please refer to official documentation here [https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli#stop-cluster-upgrades-automatically-on-api-breaking-changes Stop Upgrades]

### Troubleshoot Upgrade Failures
Please refer to the links in the section **Troubleshoot upgrade operations** in the document - https://learn.microsoft.com/en-us/troubleshoot/azure/azure-kubernetes/welcome-azure-kubernetes <br />
- Control Plane Upgrade failure
- Node failure
- Application pod failure

## AKS Cluster-level Blue-Green set-up** <br />
AKS Cluster-level Blue-Green set-up is the safest option to perform the AKS upgrades. This option gives a better ability to validate the functioning of the applications post an upgrade and it also provides an easy way to perform the roll-back. This option may prove expensive from a cost perspective. <br /><br />
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
