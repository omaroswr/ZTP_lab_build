# Installing another workload cluster: 

This installation has a couple of things that are being done differently: 

1) the IP address for the the cluster is being hardcoded in the siteConfig (instead of DHCP allocation)
2) Extra manifest use is being demonstrated. All default extraManifests are being excluded, and a manifest for chrony configuration through machineConfig is being added. 

## Create VM and DNS Entries:

```
kcli create vm -P start=False -P uefi_legacy=true -P plan=hub -P memory=24000 -P numcpus=12 -P disks=[200,200] -P nets=['{"name": "5gdeploymentlab", "mac": "aa:aa:aa:aa:05:01"}'] -P uuid=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaa0501 -P name=sno3
cp /opt/dnsmasq/include.d/sno2.ipv4 /opt/dnsmasq/include.d/sno3.ipv4
sed -i s/sno2/sno3/g /opt/dnsmasq/include.d/sno3.ipv4
sed -i s/40/42/g /opt/dnsmasq/include.d/sno3.ipv4
```

Restart DNS for new entries to take affect: 
```
systemctl restart dnsmasq-virt
```

To see the VMs: 

```
kcli list vm
```

> +----------------+--------+----------------+----------------------------------------------------+------+---------+<br>
> |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;Status&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ip&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Source&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;Plan&nbsp;|&nbsp;Profile&nbsp;|<br>
> +----------------+--------+----------------+----------------------------------------------------+------+---------+<br>
> |&nbsp;hub-ctlplane-0&nbsp;|&nbsp;&nbsp;&nbsp;up&nbsp;&nbsp;&nbsp;|&nbsp;192.168.125.20&nbsp;|&nbsp;rhcos-414.92.202310170514-0-openstack.x86_64.qcow2&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>                         
> |&nbsp;hub-ctlplane-1&nbsp;|&nbsp;&nbsp;&nbsp;up&nbsp;&nbsp;&nbsp;|&nbsp;192.168.125.21&nbsp;|&nbsp;rhcos-414.92.202310170514-0-openstack.x86_64.qcow2&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>                         
> |&nbsp;hub-ctlplane-2&nbsp;|&nbsp;&nbsp;&nbsp;up&nbsp;&nbsp;&nbsp;|&nbsp;192.168.125.22&nbsp;|&nbsp;rhcos-414.92.202310170514-0-openstack.x86_64.qcow2&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>
> |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sno1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;up&nbsp;&nbsp;&nbsp;|&nbsp;192.168.125.30&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>
> |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;up&nbsp;&nbsp;&nbsp;|&nbsp;192.168.125.40&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>
> |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sno2w&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;down&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>
> |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sno3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;down&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;hub&nbsp;&nbsp;|&nbsp;&nbsp;kvirt&nbsp;&nbsp;|<br>
> +----------------+--------+----------------+----------------------------------------------------+------+---------+<br>

## Enable secrete for SNO3 

Secrets were previously created in [this](https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_1_siteconfig_sno2.md 
) lab. So it just needs to be uncommented or added (as the case may be) to the kustomization file:

```
sed -i "s/#  - sno3/  - sno3/g" ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/secrets/kustomization.yaml
```

## Create Site-Config for SNO3: 

```
cat << EOF > ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/sno3-extra-manifest/99-chrony.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 99-master-chrony
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
        - contents:
            compression: ""
            source: data:,driftfile%20%2Fvar%2Flib%2Fchrony%2Fdrift%0Amakestep%201.0%203%0Aserver%20192.168.125.1.1%20iburst%0A
          mode: 420
          overwrite: true
          path: /etc/chrony.conf
EOF
```

```
cat << EOF > ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/sno3-extra-manifest/vpattern.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: patterns-operator
  namespace: openshift-operators
  labels:
    operators.coreos.com/patterns-operator.openshift-operators: ""
spec:
  channel: fast
  installPlanApproval: Automatic
  name: patterns-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
EOF
```

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/sno3.yaml
---
apiVersion: ran.openshift.io/v1
kind: SiteConfig
metadata:
  name: "sno3"
  namespace: "5glab"
spec:
  # The base domain used by our SNOs
  baseDomain: "5g-deployment.lab"
  # The secret name of the secret containing the pull secret for our disconnected registry
  pullSecretRef:
    name: "disconnected-registry-pull-secret"
  # The OCP release we will be deploying otherwise specified (this can be configured per cluster as well)
  clusterImageSetNameRef: "active-ocp-version"
  # The ssh public key that will be injected into our SNOs authorized_keys
  sshPublicKey: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC5pFKFLOuxrd9Q/TRu9sRtwGg2PV+kl2MHzBIGUhCcR0LuBJk62XG9tQWPQYTQ3ZUBKb6pRTqPXg+cDu5FmcpTwAKzqgUb6ArnjECxLJzJvWieBJ7k45QzhlZPeiN2Omik5bo7uM/P1YIo5pTUdVk5wJjaMOb7Xkcmbjc7r22xY54cce2Wb7B1QDtLWJkq++eJHSX2GlEjfxSlEvQzTN7m2N5pmoZtaXpLKcbOqtuSQSVKC4XPgb57hgEs/ZZy/LbGGHZyLAW5Tqfk1JCTFGm6Q+oOd3wAOF1SdUxM7frdrN3UOB12u/E6YuAx3fDvoNZvcrCYEpjkfrsjU91oz78aETZV43hOK9NWCOhdX5djA7G35/EMn1ifanVoHG34GwNuzMdkb7KdYQUztvsXIC792E2XzWfginFZha6kORngokZ2DwrzFj3wgvmVyNXyEOqhwi6LmlsYdKxEvUtiYhdISvh2Y9GPrFcJ5DanXe7NVAKXe5CyERjBnxWktqAPBzXJa36FKIlkeVF5G+NWgufC6ZWkDCD98VZDiPP9sSgqZF8bSR4l4/vxxAW4knKIZv11VX77Sa1qZOR9Ml12t5pNGT7wDlSOiDqr5EWsEexga/2s/t9itvfzhcWKt+k66jd8tdws2dw6+8JYJeiBbU63HBjxCX+vCVZASrNBjiXhFw=="
  clusters:
  - clusterName: "sno3"
    # The sdn plugin that will be used
    networkType: "OVNKubernetes"
    # All Composable capabilities removed except required for telco
    installConfigOverrides:  "{\"capabilities\":{\"baselineCapabilitySet\": \"None\", \"additionalEnabledCapabilities\": [ \"marketplace\", \"NodeTuning\" ] }}"
    extraManifestPath: site-group-1/sno3-extra-manifest
    extraManifests:
      filter:
        inclusionDefault: exclude
        include:
          - 99-chrony.yaml
          # - vpattern.yaml
    # Cluster labels (this will be used by RHACM)
    clusterLabels:
      common: "ocp414"
      syed : true 
    # Pod's SDN network range
    clusterNetwork:
      - cidr: "10.128.0.0/14"
        hostPrefix: 23
    # Network range where the SNO is connected
    machineNetwork:
      - cidr: "192.168.125.0/24"
    # Services SDN network range
    serviceNetwork:
      - "172.30.0.0/16"
    cpuPartitioningMode: AllNodes
    # If deploying 3-node, then apiVIP and ingressVIP will need to be defined as below
    # apiVIP: 192.168.125.42
    # If using dualstack, add IPv6 apiVIP here: (still keep the above line that has only IPv4. We need both!)
    # apiVIPs:
    #  - 192.168.125.42
    #  - fd00:2023:61::4
    # ingressVIP: 192.168.125.42
    # If using dualstack, add IPv6 IngressVIP here: (still keep the above line that has only IPv4. We need both!)
    #ingressVIPs:
    #  - 192.168.125.42
    #  - fd00:2023:61::5
    additionalNTPSources:
      - infra.5g-deployment.lab
    holdInstallation: false
    nodes:
      - hostName: "sno3.5g-deployment.lab"
        role: "master"
        # We can add custom labels to our nodes, these will be added once the node joins the cluster
        nodeLabels:
          5gran.lab/my-custom-label: ""
        bmcAddress: "redfish-virtualmedia://192.168.125.1:9000/redfish/v1/Systems/local/sno3"
        # The secret name of the secret containing the bmc credentials for our bare metal node
        bmcCredentialsName:
          name: "sno3-bmc-credentials"
        # The MAC Address of the NIC from the bare metal node connected to the machineNetwork
        bootMACAddress: "AA:AA:AA:AA:05:01"
        bootMode: "UEFI"
        rootDeviceHints:
          deviceName: /dev/vda
        nodeNetwork:
          interfaces:
            - name: enp1s0
              macAddress: "AA:AA:AA:AA:05:01"
          config:
            interfaces:
              - name: enp1s0
                type: ethernet
                state: up
                ipv4:
                  address:
                  - ip: 192.168.125.42
                    prefix-length: 24
                  enabled: true
                  dhcp: true
                ipv6:
                  enabled: false
EOF
```

## Add SNO3 to the kustomization files:

```
sed -i "s/generators\:/generators\:\n  - sno3\.yaml/g" ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/kustomization.yaml
sed -i "s/generators\:/generators\:\n  - site-group-1\/sno3\.yaml/g" ~/5g-deployment-lab/ztp-repository/clusters/kustomization.yaml
```

## Push to Git to kick off the installation:

```
cd ~/5g-deployment-lab/ztp-repository
git add --all
git commit -m 'Added SNO3  Site information'
git push origin main
```

## Monitor instlalation progress: 

Use the following script to monitor installation progress:

```
while true; do echo "################################"; echo "# `date` #"; echo "################################"; echo "---------------- BMH ----------------"; oc get bmh -A; echo "---------------- AgentClusterInstall ----------------"; oc get agentclusterinstall -A; echo "---------------- Clusters ----------------"; oc get managedclusters sno3 --show-labels; echo "---------------- Policies ----------------"; oc get policy -A; echo "---------------- CGU ----------------"; oc get cgu -A; sleep 15; done
```

## Check machineconfig:

When the cluster is done installing, check if the extraManifest machineconfig took effect: 

```
ssh core@192.168.125.42 -i ~/.ssh/snokey "cat /etc/chrony.conf "
```
You will see:

> driftfile /var/lib/chrony/drift<br>
> makestep 1.0 3<br>
> server 192.168.125.1.1 iburst<br>
