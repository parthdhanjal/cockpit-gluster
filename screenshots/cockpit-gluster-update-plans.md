# Cockpit Gluster Update!

Hey! We are adding a few features to cockpit-gluster.
Adding a few mockups along with it for review and comments.
The mockups are rather rough and just depict the solution a little roughly.
The CSS Styling will be inline with patternfly as before

* Adding more options and actions to Gluster Management Page
   Adding an option to start/stop volumes, setting gluster volume options.

![Gluster Management Dashboard Image](GlusterManagementUpdates.png?raw=true "Gluster Management Dashboard")

* Adding multi-host support. Until now you could expand only 3 hosts at a time through the expand cluster wizard. Planning to change it to a multi-host support.
   Now there will be only a single field as default in the first step, and a Add Host button which will add more hosts as needed

![Expand Cluster Multiple Add Host](ExpandClusterMultipleAddHosts.png?raw=true "Expand Cluster Multiple Add Host")

* Volume Creation is optional
   All fields are inactive when the Add Volumes checkbox is unchecked

![Expand Cluster Optional Volume Creation Unchecked](ExpandClusterOptionalVolumeCreationUnchecked.png?raw=true "Expand Cluster Optional Volume Creation Unchecked")

* When the volume is checked, the fields are activated and multiple volumes can be added similar to the multiple hosts.

![Expand Cluster Optional Volume Creation Checked](ExpandClusterOptionalVolumeCreationChecked.png?raw=true "Expand Cluster Optional Volume Creation Checked")

* EC and Replica volume support.
	Since there are multiple hosts and volume will not be created on all hosts like before. Providing options to select the hosts on which the replica volume need to be created through a multiple select checkbox. In case of EC Volumes, Taking inputs as Redundancy Count and Disperse Data Count and also selecting the hosts on which the bricks for the volumes will be formed

![Expand Cluster Multiple Host Selection and Disperse Volumes](ExpandClusterMultipleHostSelectionandDisperseVolumes.png?raw=true "Expand Cluster Multiple Host Selection and Disperse Volumes")

Now taking information from volumes tab we will automatically list bricks on each host according to the volume configurations mentioned in the volumes tab. The user won’t be able to add/delete bricks on the bricks tab. In case there are any changes that need to be made after reviewing the bricks tab, the user will have to make the changes under the volumes tab again.
