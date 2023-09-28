# Upgrade Guidance for Azure Kubernetes Service

The article explains the considerations and best practices related to upgrading your Azure Kubernetes Service clusters. <br /><br />

## Introduction

Upgrading your Azure Kubernetes Service (AKS) cluster is an essential task to ensure that your applications are running on the latest and most secure version of Kubernetes. However, upgrading an AKS cluster can be a complex process that requires careful planning and execution to avoid downtime or data loss. In this guide, we will provide guidance on how to upgrade your AKS cluster to the latest version, including best practices and tips to ensure a smooth and successful upgrade process.

Upgrading an AKS cluster offers several benefits, including:

1. Security: Upgrading to the latest version of Kubernetes ensures that your cluster is running on the most secure version of the platform, with the latest security patches and bug fixes.
2. Performance: Newer versions of Kubernetes often come with performance improvements, such as faster networking and more efficient resource utilization, which can improve the overall performance of your applications.
3. Stability: Upgrading to a newer version of Kubernetes can help improve the stability of your cluster by fixing bugs and addressing issues that may have been present in previous versions.
4. Feature enhancements: Newer versions of Kubernetes often come with new features and enhancements that can help improve the functionality and capabilities of your cluster.
5. Support: Running your AKS cluster on a supported version of Kubernetes ensures that you can receive support from the Kubernetes community and Microsoft, which can be critical in the event of issues or outages.

Note: This guide is opinionated and based on a general variety of customers. There could be specific scenarios where this guide may not be able to help you put all the best practices in place. We recommend you contact Microsoft Specialists and Support for up-to-date and contextual guidance. 
This guide is divided into three main sections 
..* - Prepare for Upgrade - Will involve guidance around planning for the AKS upgrade
..* - Execute the Upgrade - This will be list of things to consider when upgrading
..* - Post Upgrade - This will be set of steps to validate a successful upgrade as well as measure impact to your application.
 
## Prepare for Upgrade
Here are the things that you will consider when preparing for an upgrade

Upgrade Type - Manual vs Automated
Upgrade Frequency - Stable, Rapid, Patch, etc. 
Upgrade Duration - Node Surge, Node pool based upgrade, maintenance window, etc. 
Upgrade Propogation across Environment

Identify deprecated APIs and impact

## Execute the Upgrade
When you are ready for executing the upgrades - please ensure the following things. 

### Pre-Upgrade Validation 
- IP addresses available - Azure CNI
- Pod Resiliency and placement -
- Stateful Sets
- Downtime


### Upgrade Steps 

#### Manual

- Downtime 
- Commands
- MOnitor
- App changes deployment

#### Automated 

- Downtime
- MOnitor
- App changes deployment

## Post Upgrade
When you are ready for executing the upgrades - please ensure the following things. 

### Checks
- Version Checks  
- Application Health Checks

### Troubleshoot Upgrade Failures

- Control Plane Upgrade failure
- Node failure
- Application pod failure

### Propogation to next environment

- Maintenance Schedule
- Infra As Code changes 

