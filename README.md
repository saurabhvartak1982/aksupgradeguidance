# aksupgradeguidance

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



## Day-2 set-up for AKS cluster upgrades <br />
###	AKS Release calendar <br />
####		Cluster-wise planning of upgrades of each environment basis the AKS Release calendar- refer sample excel <br />
###	Check for pre-requisites and breaking changes <br />
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
			
			Kubectl get nnc -- <br />

