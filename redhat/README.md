# SAP HA Interface Script Connector for Pacemaker Based Cluster Setups
## Purpose
This repo provides the componnents to integrate a SAP environment managed by a [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker) based cluster with the HA Interface provided by SAP.

This allows that SAP instance which are manged by a cluster to be controlled by SAP management tools like the SAP MMC. Also SAP tools like SUM are able to notifiy the cluster that  during an update of an SAP instance activieties that can result in a restart of a SAP innstance which should not be acted upon by the cluster.

## Resources

[Using sap_vendor_cluster_connector for interaction between cluster framework and sapstartsrv](https://blogs.sap.com/2014/05/08/using-sapvendorclusterconnector-for-interaction-between-cluster-framework-and-sapstartsrv/)

[Manual Page of sap_vendor_cluster_connector](https://wiki.scn.sap.com/wiki/display/ATopics/Manual+Page+of+sap_vendor_cluster_connector)

[Important Resources about the HA-Interface Certification for Partners](https://wiki.scn.sap.com/wiki/display/SI/Important+Resources+about+the+HA-Interface+Certification+for+Partners)

[SAP Note 1693245 - SAP HA Script Connector Library](https://launchpad.support.sap.com/#/notes/1693245)

[SAP Note 1822055 - Enhanced SAP HA library interface](https://launchpad.support.sap.com/#/notes/1822055)

[SAP Note 2464065 - Check of automatic maintenance mode for HA solutions](https://launchpad.support.sap.com/#/notes/2464065)

## Installation
  * copy the `sap_cluster_connector` script to `/usr/bin/`
  * create the directory `/usr/share/sap_cluster_connector/`
  * copy the `run_checks` script to `/usr/share/sap_cluster_connector/`
  * copy the `checks` directory and all its contents to `/usr/share/sap_cluster_connector/`
  * make sure `/usr/bin/sap_cluster_connector`, `/usr/share/sap_cluster_connector/run_checks` and all scripts in `/usr/share/sap_cluster_connector/checks/` are executable by all users:  
```
# chmod a+x /usr/bin/sap_cluster_connector
# chmod a+x /usr/share/sap_cluster_connector/run_checks
# chmod a+x /usr/share/sap_cluster_connector/checks/*
```
 
## Cluster Configuration
  * activate [record-pending](https://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Pacemaker_Explained/_resource_operations.html) resource operation option:  
  pcs (RHEL): `# pcs resource op defaults record-pending=true`  
  crmsh (SLES): `# crm configure op_defaults record-pending=true`  
  **Note:** on pacemaker 2.x and some pacemaker 1.x versions shipped with some Linux distributions`record-pending` is already activated by default. Please check the release notes of your pacemaker version to se if this is the case or if `record-pending` needs to be activated manually.

  * add SIDadm user to "haclient" group on each node:  
  ```
  # usermod -a -G haclient rh2adm
  ```

## SAP Configuration
  * make sure that the [SAP HA Script Connector Library](https://launchpad.support.sap.com/#/notes/1693245) (saphascriptco.so) is available as part of the SAP Kernel of the SAP installation

  * add the following to the instance profile of each SAP instance that should be managed via the SAP HA interface (for example /sapmnt/RH2/profile/RH2_ASCS00_rh2ascs):  
  ```
  service/halib = $(DIR_CT_RUN)/saphascriptco.so
  service/halib_cluster_connector = /usr/bin/sap_cluster_connector
  ```

  * restart the Instance (including the sapstartsrv for that instance)

## Usage
The cluster connector is normaly called automatically in case an action on a SAP insance is performed via the SAP management tools that requires some interaction with the cluster.

To verify that the cluster connector is working correctly it is also possible to run the sap_cluster_connector script manually or to trigger its execution via `sapcontrol`.

### Examples for running sap_cluster_connector directly
Do a minor check, if the cluster framework command line interface is available:
```
# /usr/bin/sap_cluster_connector init
```
Get some basic information about the cluster product used:
```
#/usr/bin/sap_cluster_connector gvi --out /tmp/gvi.txt
```
Look-up the current cluster node and all possible cluster nodes to run the cluster resource named rsc_sap_C11_D02:
```
# /usr/bin/sap_cluster_connector lsn --out /tmp/lsn.txt --res rsc_sap_C11_D02
```
Look-up the SAP instance number 02 of the SAP system C11 and return the cluster resource name:
```
# /usr/bin/sap_cluster_connector lsr --out /tmp/lsr.txt --sid C11 --ino 02
```
Start the cluster resource rsc_sap_C11_D02:
```
# /usr/bin/sap_cluster_connector fra --res rsc_sap_C11_D02 --act start
```
Check, if the cluster action to start the SAP instance for system C11 and instance number 02 is already in progress:
```
# /usr/bin/sap_cluster_connector cpa --res rsc_sap_C11_D02 --act start
```
Move the cluster resource rsc_sap_C11_D02 to the cluster node node2:
```
# /usr/bin/sap_cluster_connector fra --res rsc_sap_C11_D02 --act migrate --node node2
```
### Examples for calling sap_cluster_connector via sapcontrol
Perform a check of the HA configuration:
```
# /usr/sap/hostctrl/exe/sapcontrol -nr 29 -function HACheckConfig
```
Trigger the failover of a SAP instance to another cluster node:
```
# /usr/sap/hostctrl/exe/sapcontrol -nr 20 -function HAFailoverToNode
```

## TODO
  * implement function to activate maintenance mode (sww)
  * add functionality to also read information about HA product from config file
  * add check for record-pending option
  * code cleanup

