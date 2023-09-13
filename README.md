# aksupgradeguidance
## 1. Kubernetes cluster logical diagram
Logical diagram here featuring Control Plane and Data Plane

## 2. Types of AKS related upgrades
AKS version upgrades for Control Plane and Data Plane <br />
Node Image Upgrades for Data Plane Nodes <br />
Node OS patches for the Data Plane Nodes <br />
AKS Control Plane maintenance --? <br /><br />

## 3. Ensuring application availability during upgrades
### a. Kubernetes/AKS-based
Node Surge <br />
Pod Disruption Budgets <br />
NodePool-level Blue-Green set-up<br /><br />
### b. Architecture patterns-based
AKS Cluster-level Blue-Green set-up<br /><br />

## 4. Available AKS upgrades related information
AKS Upgrade Release calendar ... need to detail what info is provided by this document? <br />
AKS Upgrade GitHub release page ... need to detail what info is provided by this document? <br />
Microsoft Defender Notifications ... need to detail what info is provided by this document? <br />
Availability of a patch upgrade ---? ... need to detail what info is provided by this document? <br />
az-cli (az aks get-upgrades, az aks nodepool get-upgrades) ... need to detail what info is provided by this document? <br />

## 5. Ways to upgrade an AKS cluster
### a. Manual
####     Check for available versions <br />
#####      az aks get-upgrades ---- for fetching the available upgrades  <br />
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
####      Ensure that the IP address availability is enough to cater to the Node Surge
####      If AKS cluster-level Blue-Green set-up is in place, ensure that the above mentioned pre-requsites are in-place for both the clusters <br />
###     Check for breaking changes 
####      Check for any breaking changes which may result as a result of the upgrade. This can be done using one of the below 2 methods: <br />
#####       a. <b>Kubernetes API Deprecations</b> view in the <b>Diagnostic Setting --> Create, Upgrade, Delete and Scale <b /> of the AKS cluster page
#####       b. Raising an advisory support case with Microsoft
###     Sequential upgrades of the AKS clusters different environments basis the schedule defined in AKS_Cluster_Upgrade_Plan.xlsx <br />
###     Post-upgrade alert <br />
###     Post-upgrade update of the AKS_Cluster_Upgrade_Plan.xlsx <br />
###     Automated/Manual Sanity/Perf Testing post an upgrade -  basis the environment <br />

## LTS option




			
			Kubectl get nnc -- <br />

