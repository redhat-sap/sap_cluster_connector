# SAP HA Interface Script Connector for Pacemaker Based Cluster Setups
## Purpose
This repo provides the componnents to integrate a SAP environment managed by a [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker) based cluster with the HA Interface provided by SAP.

This allows that SAP instance which are manged by a cluster to be controlled by SAP management tools like the SAP MMC. Also SAP tools like SUM are able to notifiy the cluster that  during an update of an SAP instance activieties that can result in a restart of a SAP innstance which should not be acted upon by the cluster.

## Resources

[Connecting startsapsrv and cluster frameworks using the components saphascriptco.so and sap_vendor_cluster_connector](https://blogs.sap.com/2016/01/18/connecting-startsapsrv-and-cluster-frameworks-using-the-components-saphascriptcoso-and-sapvendorclusterconnector/)

[Important Resources about the HA-Interface Certification for Partners](https://wiki.scn.sap.com/wiki/display/SI/Important+Resources+about+the+HA-Interface+Certification+for+Partners)

[Manual Page of sap_vendor_cluster_connector](https://wiki.scn.sap.com/wiki/display/ATopics/Manual+Page+of+sap_vendor_cluster_connector)

[SAP Note 1693245 - SAP HA Script Connector Library](https://launchpad.support.sap.com/#/notes/1693245)

[SAP Note 1822055 - Enhanced SAP HA library interface](https://launchpad.support.sap.com/#/notes/1822055)

[SAP Note 2464065 - Check of automatic maintenance mode for HA solutions](https://launchpad.support.sap.com/#/notes/2464065)

## Installation
  * copy the `sap_cluster_connector` script to `/usr/bin/`
  * create the directory `/usr/share/sap_cluster_connector/`
  * copy the `run_checks` script to `/usr/share/sap_cluster_connector/`
  * copy the `checks` directory and all its contents to `/usr/share/sap_cluster_connector/`
 
## Cluster Configuration
  * activate [record-pending](https://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Pacemaker_Explained/_resource_operations.html) resource operation:  
  ```
  # pcs resource op defaults record-pending=true
  ```

  * add SIDadm user to "haclient" group on each node:  
  ```
  # usermod -a -G haclient rh2adm
  ```
## SAP Configuration
  * make sure that the [SAP HA Script Connector Library](https://launchpad.support.sap.com/#/notes/1693245) (saphascriptco.so) is available as part of the SAP Kernel of the SAP installation

  * add the following to the instance profile of each SAP instance that should be managed via the SAP HA interface (for example /sapmnt/RH2/profile/RH2_ASCS00_rh2ascs):  
  ```
  service/halib = $(DIR_CT_RUN)/saphascriptco.so
  service/halib_cluster_connector = /usr/bin/sap_redhat_cluster_connector
  ```

  * restart the Instance (including the sapstartsrv for that instance)
