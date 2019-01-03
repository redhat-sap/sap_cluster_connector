# SAP HA Interface Script Connector for Pacemaker Based Cluster Setups
## Purpose
This repo provides the componnents to integrate a SAP environment managed by a [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker) based cluster with the HA Interface provided by SAP.

## File structure
  * `redhat` contains the updated version of the cluster connector which should work with any [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker) based cluster setup (it has only been tested with the RHEL HA Add-On for RHEL 7 so far)
  * `sapha-1.1.0` contains the original `sap_suse_cluster_connector` created by SUSE, which only works with the SLE HA Extension

