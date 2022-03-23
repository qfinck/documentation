### Release Notes

This document provides information about the Diamanti D-Series v3.3.0 (121) release for Ultima Enterprise and Ultima Enterprise with Accelerator products.

### What’s New in the Release

###### EBS Volume Support
This release supports EBS volume backaend with Ultima Enterprise in AWS. 

###### RWX Support
Diamanti CSI now supports RWX (ReadWriteMany) access for the volumes.

###### Fire-drill support for DR
This release adds Fire-drill support for applications for which DR is configured.

###### Flexible number of drives
Ultima Enterprise users can now set up their nodes with any numbers of drives (upto 4). Before this change, users had to configure exactly 4 drives for each node in the cluster.

###### DSS Class support to tune CPU consumption by Diamanti Storage software
This feature provides users the flexbility to choose resource consumption by Diamanti Storage Software based on their environment and IOPs requirements. In 3.3.0, three DSS classes are supported (small, medium, large). This feature is applicable for the Ultima Enterprise (dvx software appliance).

###### Drive Health monitoring via SNMP
Drive health notifications are added in 3.3.0 for Ultima Enterprise with Accelerator product to monitoring drive health via SNMP.

###### Automated way of cleaning up dependent PVC objects
3.3.0 release enhances clean up of dependent objects when a PVC is deleted.

###### Route metric support for multi-endpoints PODs
When running PODs with multiple network endpoints, users now have the flexibility of providing default route metric for each network endpoint in the network annotion of the POD spec. A new field, ```metric``` has been added to the annotation. As an example:
```
  annotations:
    diamanti.com/endpoint0: '{"network":"blue","perfTier":"high","metric":20}'
```

The release additionally updates software to the following newer versions:

* Kubernetes 1.19.15
* Helm 3.5.2
* Cri-o 1.18.4

### Resource Maximums

The release supports the following maximum number of resources (per node):

| Resource (per node)        | Diamanti D-Series v3.3.0 (121)   |
| ---------------------------|:--------------------------------:|
| Pods                       |110                               |
| Volumes                    |2048(total)/64(active)\*          |
| VNICs                      |63                                |
| Storage controllers        |64                                |
| Remote storage controllers |64                                |
| KVMs                       |8                                 |


The maximum number of volumes on a node is 2048. This number is equal to the sum of all mirrors of all volumes, snapshots, linked clones, etc. A snapshot is considered a volume.
The maximum number of snapshots per volume is 16.
The maximum number of linked clone volumes on a snapshot is up to the maximum number of volumes on a node.
The number of volumes exposed to a host is 64 (active volumes). The number of volumes a node can serve as a target is 64. Therefore, at any time, a node can expose 64 volumes to the host, as well as serve 64 volumes as a target.
For example, in a three-node cluster:
* The number of simple volumes (single mirror): 6K
* The number of 2-way mirrored volumes: 3K
* The number of 3-way mirrored volumes: 2K


### Release Details

*Note* Upgrade to this release is applicable only if you are running Docker runtime engine. Upcoming 3.3.1 release will support upgrade for Cri-o runtime engine. 

Note that PCIe NVME drives are *not* hot-pluggable. Diamanti recommends that you do not swap drives installed on a node after volumes are created on the drive. If a drive malfunctions, contact Diamanti Support.

### Release Requirements

Diamanti v3.3.0 (121) requires Diamanti D-Series OS Release 7.6.0-26.

#### Upgrade Matrix

Use the following matrix to determine whether an upgrade is supported from your current running release to Diamanti v3.3.0 (121).

| Running Release      | New Release    | Supported |
| :-------------------:|:--------------:|:---------:|
| v3.2.1               | v3.3.0         | Yes       |
| v3.2.0               | v3.3.0         | Yes       |
| v3.1.2               | v3.3.0         | Yes       |
| v3.1.1               | v3.3.0         | Yes       |
| v3.0.1               | v3.3.0         | No        |
| v3.0.0               | v3.3.0         | No        |
| v2.4.\*              | v3.3.0         | No        |


### Upgrade Steps

1.Download the upgrade bundle from Diamanti Central.

2.Identify the master node in the cluster:

```
    dctl -o json cluster status | grep master
```

3.Copy the upgrade file to the ```/home/diamanti``` directory on the master node.


4.Untar the upgrade script on the master node of the cluster:

```
    tar -zxvf diamanti-upgrade-rpms-3.3.0-121.tgz
```

5.Copy the ```upgrade_node-3.3.0.sh``` script to a non-master etcd node in the cluster. On the master node and the non- master etcd node, ensure that the upgrade_node-3.3.0.sh script is executable. If the script isn't executable, modify its permissions to make it executable.

```
    chmod a+x upgrade_node-3.3.0.sh
```
  
6.On the non-master etcd node, run the upgrade command to apply the all pre-upgrade patches cluster-wide. Run this command once per cluster.

```
    dctl login
    ./upgrade_node-3.3.0.sh -i <master node hostname>
```

This command takes a few minutes to run.


7.After you apply the pre-upgrade patches, you can upgrade the master node.

```
    dctl login
    ./upgrade_node-3.3.0.sh -u <master node hostname>
```

This process takes roughly 30 minutes per node.

8.After you upgrade the master node, upgrade the non-master etcd nodes next, and then the worker nodes. Upgrade one node at a time until the rest of the cluster is upgraded. Ensure that you run the upgrade_node-3.3.0.sh script from the master node to upgrade the rest of the cluster.

```
    dctl login
    ./upgrade_node-3.3.0.sh -u <node hostname>
```
  

### Known Issues

  

This section lists the known issues for the Diamanti D-Series v3.3.0 (121) release.

  

### On clusters enabled with kubevirt feature, 2Mi Huge Pages do not clean up after upgrade to 3.3.0 (DWS-8171)

  

**Description:** If kubevirt feature is enabled on build (ver < 3.3.0) then during cluster upgrade, node status may show hugepages allocation as enabled. This only affects clusters with kubevirt feature enabled.

  

**Workaround:** Before upgrading to 3.3.0, delete manually hugepages entry from ```/etc/sysctl.conf``` on all cluster nodes and then upgrade the cluster.

  

### diamanti-kv-docker-core rpm is not updated when upgrading clusters with kubevirt feature enabled (DWS-8170)

  

**Description:** This is applicable to only those clusters that have kubevirt feature enabled. The diamanti-kv-docker-core rpm is not updated automatically when upgrading Ultima cluster.

  

**Workaround:** If the kubevirt feature is enabled on your cluster, install the diamanti-kv-docker-core rpm before upgrading the cluster to 3.3.0 using the following steps:

1. Download the diamanti-kv-docker-core rpm from the Diamanti support portal to the cluster ```Master``` node

2. Run the following dctl command to update the diamanti-kv-docker-core rpm on all the nodes in the cluster:

```

dctl cluster update software -i < diamanti-kv-docker rpm path> --no-reboot --no-drain

```

### Known Issues

This section lists the known issues for the Diamanti D-Series v3.3.0 (121) release.

### On clusters enabled with kubevirt feature, 2Mi Huge Pages do not clean up after upgrade to 3.3.0 (DWS-8171)

**Description:** If kubevirt feature is enabled on build (ver < 3.3.0) then during cluster upgrade, node status may show hugepages allocation as enabled. This only affects clusters with kubevirt feature enabled.

**Workaround:** Before upgrading to 3.3.0, delete manually hugepages entry from ```/etc/sysctl.conf``` on all cluster nodes and then upgrade the cluster.

### diamanti-kv-docker-core rpm is not updated when upgrading clusters with kubevirt feature enabled (DWS-8170)

**Description:** This is applicable to only those clusters that have kubevirt feature enabled. The diamanti-kv-docker-core rpm is not updated automatically when upgrading Ultima cluster.

**Workaround:** If the kubevirt feature is enabled on your cluster, install the diamanti-kv-docker-core rpm before upgrading the cluster to 3.3.0 using the following steps:
1. Download the diamanti-kv-docker-core rpm from the Diamanti support portal to the cluster ```Master``` node
2. Run the following dctl command to update the diamanti-kv-docker-core rpm on all the nodes in the cluster:
```
    dctl cluster update software -i < diamanti-kv-docker rpm path> --no-reboot --no-drain
```

### Legal Notices
Publication Date: This document was published on Nov 1, 2021.
Publication Number: DM-RN-20211101-01

#### Copyright
Copyright © 2016-2021, Diamanti. All rights reserved.

Diamanti believes the information it furnishes to be accurate and reliable. However, Diamanti assumes no responsibility
for the use of this information, nor any infringement of patents or other rights of third parties which may result from its use. No license is granted by implication or otherwise under any patent, copyright, or other intellectual property right of Diamanti except as specifically described by applicable user licenses. Diamanti reserves the right to change specifications at any time without notice.

### Trademarks
Diamanti and the Diamanti GUI are trademarks or service marks of Diamanti, in the U.S. and other countries, and may
not be used without Diamanti's express written consent.
All other product and company names herein may be trademarks of their respective owners.

