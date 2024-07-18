[toc]

# Changes compared to original lab: 

## Install Butane: 

On Bastion, before disconnecting: 
```
curl https://mirror.openshift.com/pub/openshift-v4/clients/butane/latest/butane --output butane
mv butane /usr/bin/
chmod u+x /usr/bin/butane
```

Note:  OpenShift Container Platform supports the addition of a single partition to attach storage to either the /var directory or a subdirectory of /var. For example:

Important Directories: 

| location | purpose | 
|--------|-------------|
/var/lib/containers | Used to store container imaes and data | 
/var/lib/etcd | Used to store etcd data | 
/var | any data that user may want to keep segregated | 

## In hub.yaml:

Instead of: 
```
 disks:
 - size: 200
 - size: 300
```
Use: 
```
 disks:
 - size: 16000
 - size: 800
```

## In agent-config.yaml:

Add "rootDeviceHint" to ensure that partitioned disk is consistently used: 

```
apiVersion: v1alpha1
metadata:
  name: hub
rendezvousIP: 192.168.125.100
hosts:
  - hostname: sno
    rootDeviceHints:
      deviceName: "/dev/vda"
    interfaces:
     - name: eth0
       macAddress: 52:54:00:35:bb:80
    networkConfig:
      interfaces:
      - name: eth0
        state: up
        ipv4:
          address:
          - ip: 192.168.125.100
            prefix-length: 24
          enabled: true
          dhcp: false
      dns-resolver:
        config:
          server:
            - 192.168.125.1
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-address: 192.168.125.1
            table-id: 254
            next-hop-interface: eth0
```

# Steps to follow before beuidling ISO:

Before beuilding ISO, a machineconfig has to be placed in the correct location so it pikcs up the partiioning instructions. This machine config has built using butane. So steps will be: 
1) create butane file 
2) use butane utililty to create maching config, and put it under `~/abi/openshift` directory 
3) now build the ISO as usual

## 1) Butane File:

```
variant: openshift
version: 4.15.0
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 98-var-partition
storage:
  disks:
  - device: /dev/disk/by-path/pci-0000:04:00.0
    partitions:
      - number: 1
        should_exist: true
      - number: 2
        should_exist: true
      - number: 3
        should_exist: true
      - number: 4
        resize: true
        size_mib: 460000
      - label: var-lib-containers
        number: 5
        size_mib: 460000
        start_mib: 0
        wipe_partition_entry: true
      - label: var-lib-etcd
        number: 6
        size_mib: 460000
        start_mib: 0
        wipe_partition_entry: true
      - label: var-lib-prometheus-data
        number: 7
        size_mib: 0
        start_mib: 0
        wipe_partition_entry: true
  filesystems:
    - device: /dev/disk/by-partlabel/var-lib-containers
      format: xfs
      mount_options: [defaults, prjquota]
      path: /var/lib/containers
      wipe_filesystem: true
      with_mount_unit: true
    - device: /dev/disk/by-partlabel/var-lib-etcd
      format: xfs
      mount_options: [defaults, prjquota]
      path: /var/lib/etcd
      wipe_filesystem: true
      with_mount_unit: true
    - device: /dev/disk/by-partlabel/var-lib-prometheus-data
      format: xfs
      path: /var/lib/prometheus/data
      wipe_filesystem: true
      with_mount_unit: true
      mount_options: [defaults, prjquota]
```

## 2)  Create the machine config: 
```
butane ~/partition.bu -o ~/98-var-partition.yaml
```
Now: 
```
mkidr -p ~/abi/openshift/
mv 98-var-partition.yaml ~/abi/openshift/
```

## 3) Create the ABI image as before: 

No new step here. 
