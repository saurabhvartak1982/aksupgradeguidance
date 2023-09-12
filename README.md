# aksupgradeguidance
## Kubernetes cluster logical diagram
Logical diagram here featuring Control Plane and Data Plane

## Types of AKS related upgrades
AKS version upgrades for Control Plane and Data Plane <br />
Node Image Upgrades for Data Plane Nodes <br />
Node OS patches for the Data Plane Nodes <br />
AKS Control Plane maintenance --? <br /><br />

## Ensuring application availability during upgrades
Node Surge <br />
Pod Disruption Budgets <br />

## Available AKS upgrades related information
AKS Upgrade Release calendar <br />
AKS Upgrade GitHub release page <br />
Microsoft Defender Notifications <br />
az-cli (az aks get-upgrades, az aks nodepool get-upgrades) <br />

## Ways to upgrade an AKS cluster
Manual <br />
Automated <br />
a. Auto-upgrade channel + Maintenance Window <br />
b. GitHub Actions OR Cron Jobs <br /><br />

## Day-2 operations related to AKS
Day-2 operations comprise of monitoring and maintenance operations of the AKS cluster. AKS related upgrades are part of the maintenance operations. <br />

## Day-2 set-up for AKS cluster upgrades <
###     Cluster-wise and environment-wise planning of AKS upgrades 
####      Cluster-wise planning of upgrades of each environment basis the AKS Release calendar- refer sample excel 
###     Check for pre-requisites 
###     Check for breaking changes 
###	Sequential upgrades of different environments <br />
####		Frequency of upgrades <br />
#####			Need to check what are different persona types of the customer and suggest basis the same <br />
####		Selection of the type of upgrade (automated/manual, upgrade channel, etc. ) basis the environment <br />
###	Sanity/Perf Testing post an upgrade - Automated/Manual <br />
###	Auto-upgrades <br />
####		Cluster auto-upgrades <br />
#####			<more detailing here> <br />
####		Node OS auto-upgrades <br />
#####			<more detailing here> <br />
###	Maintenance Window <br />
####		Window for Cluster Upgrade <br />
#####			<more detailing here> <br />
####		Window for Node Upgrade <br />
#####			<more detailing here> <br />

## During the manual upgrade <br />
###	Pre-requisites <br />
####		Ensure PDBs and Node Surge settings are in place. For Zone-redundant Nodepools ensure that Node Surge is in the multiples of 3 <br />
####		Check for quota and IP address availability --? <br />
####		If Green-Blue AKS Cluster set-up is in place, ensure that the above is in-place for both the clusters <br />
####		Advisory support case with Microsoft <br />
#####			App Lens <br />
#####			Diagnostic Settings <br />
###	Check for available versions <br />
####		az aks get-upgrades ---- for fetching the available upgrades  <br />
####		az aks nodepool get-upgrades ----- for fetching the available node image upgrades only, NOT for AKS upgrades <br />
###	Check for breaking changes and fix the same--? <br />
####		Check using the Release Notes <br />
####		Check using the built-in Notebooks/Dashboard <br />
###	Upgrade the Control Plane <br />
###	Upgrade the Nodepools - 1 upgrade at a time <br />
###	View upgrade events <br />
###	Verification of the upgrade which has been performed <br />
###	Sanity/Perf Testing post an upgrade - Automated/Manual <br /><br /><br />




			
			Kubectl get nnc -- <br />

