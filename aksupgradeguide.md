# Upgrade Guidance for Azure Kubernetes Service

The article explains the considerations and best practices related to upgrading your Azure Kubernetes Service clusters. <br /><br />

## Introduction

Upgrading your Azure Kubernetes Service (AKS) cluster is an essential task to ensure that your applications are running on the latest and most secure version of Kubernetes. However, upgrading an AKS cluster can be a complex process that requires careful planning and execution to avoid downtime or data loss. In this guide, we will provide guidance on how to upgrade your AKS cluster to the latest version, including best practices and tips to ensure a smooth and successful upgrade process.

Upgrading an AKS cluster offers several benefits, including:

#### 1. Security: Upgrading to the latest version of Kubernetes ensures that your cluster is running on the most secure version of the platform, with the latest security patches and bug fixes.
#### 2. Performance: Newer versions of Kubernetes often come with performance improvements, such as faster networking and more efficient resource utilization, which can improve the overall performance of your applications.
#### 3. Stability: Upgrading to a newer version of Kubernetes can help improve the stability of your cluster by fixing bugs and addressing issues that may have been present in previous versions.
#### 4. Feature enhancements: Newer versions of Kubernetes often come with new features and enhancements that can help improve the functionality and capabilities of your cluster.
#### 5. Support: Running your AKS cluster on a supported version of Kubernetes ensures that you can receive support from the Kubernetes community and Microsoft, which can be critical in the event of issues or outages.

Note: This guide is opinionated and based on a general variety of customers. There could be specific scenarios where this guide may not be able to help you put all the best practices in place. We recommend you contact Microsoft Specialists and Support for up-to-date and contextual guidance. 
This guide is divided into three main sections 
#### - Prepare for Upgrades - Will involve guidance around planning for the AKS upgrade
#### - Execute the Upgrade - This will be list of things to consider when upgrading
#### - Post Upgrade - This will be set of steps to validate a successful upgrade as well as measure impact to your application.
 
### Plan


The **first 5 sections** provide a quick primer as to what are the different concepts involved as far as Azure Kubernetes Service related upgrades are concerned.<br />
**Section 6** gives a quick intro to the **AKS related Day-2 operations** and **section 7** tries to stitch together all these concepts and provides a point-of-view as to how the AKS Cluster upgrades can be planned and executed. <br />
**Section 8** introduces an additional option of the Long Term Support (LTS) - not yet available <br /><br />
To prevent duplication, the article relies on the official AKS or Kubernetes documentation and mentions the references to the same, wherever applicable. <br /><br />

**Kindly note - the approach depicted in this article is a personal point of view. Reader's discretion is advised** <br /><br />

## 1. Kubernetes cluster logical diagram
A Kubernetes cluster is logically divided into 2 parts: <br /><br />
**a. Control Plane:** Core Kubernetes Services <br />
**b. Data Plane (Nodes in the NodePool):** Nodes running the application workloads <br />
![Kubernetes cluster logical diagram](/images/k8sclusterarchitecturediag.png) <br />
Diagram courtesy - https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#kubernetes-cluster-architecture <br />

Detailed overview on the Control Plane can be found here - https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#control-plane <br />
Detailed overview on the Data Plane (Nodes and NodePools) can be found here - https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#nodes-and-node-pools <br /><br />
