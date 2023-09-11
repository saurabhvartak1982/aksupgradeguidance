# aksupgradeguidance

During the manual upgrade
	Pre-requisites
		Ensure PDBs and Node Surge settings are in place. For Zone-redundant Nodepools ensure that Node Surge is in the multiples of 3
		Check for quota and IP address availability --?
		If Green-Blue AKS Cluster set-up is in place, ensure that the above is in-place for both the clusters
		<Other practices as per Rishabh Saha>
		Advisory support case with Microsoft
			App Lens
			Diagnostic Settings
	Check for available versions
		az aks get-upgrades ---- for fetching the available upgrades 
		az aks nodepool get-upgrades ----- for fetching the available node image upgrades only, NOT for AKS upgrades
	Check for breaking changes and fix the same--?
		Check using the Release Notes
		Check using the built-in Notebooks/Dashboard
	Upgrade the Control Plane
	Upgrade the Nodepools - 1 upgrade at a time
	View upgrade events
	Verification of the upgrade which has been performed
	Sanity/Perf Testing post an upgrade - Automated/Manual



Day-2 set-up for AKS cluster upgrades
	AKS Release calendar
		Cluster-wise planning of upgrades of each environment basis the AKS Release calendar- refer sample excel
	Check for pre-requisites and breaking changes 
	Sequential upgrades of different environments
		Frequency of upgrades
			Need to check what are different persona types of the customer and suggest basis the same
		Selection of the type of upgrade (automated/manual, upgrade channel, etc. ) basis the environment
	Sanity/Perf Testing post an upgrade - Automated/Manual
	Auto-upgrades
		Cluster auto-upgrades
			<more detailing here>
		Node OS auto-upgrades
			<more detailing here>
	Maintenance Window
		Window for Cluster Upgrade
			<more detailing here>
		Window for Node Upgrade
			<more detailing here>
			
			Kubectl get nnc -- 

