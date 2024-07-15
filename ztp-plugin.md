# Understand the ZTP Plugin:

Purpose: To understand the use of Red Hat OpenShift ZTP Plug-in. 

## Install the ZTP Plugin Locally:

Create Directory structure first: 

```
sudo -i
mkdir -p ~/ztp/plugin/
```
### Extract ZTP Plugin:: 

```
podman cp $(podman create --name policgentool --rm quay.io/openshift-kni/ztp-site-generator:latest):/kustomize/plugin/ran.openshift.io ~/ztp/plugin/
podman rm -f policgentool
```

**NOTE** The prduction person of the plugin that you should point to, and use, is on `registry.redhat.io`. So the above command could have pulled `egistry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.14`. However, to by pass use of pull secret and authentication, the public unrestricted version on quay.io is used in this example. 

## Create SiteConfig:

First create a directory structure: 
```
mkdir -p ~/ztp/5glab
```

Use the followign as a sample siteconfig file to use: 

**NOTE** there aren't any additional `extraManifsts` being passed to this file, and the use of default `extraManifests` is allowed. 

```
cat <<EOF > ~/ztp/5glab/siteconfig.yaml
---
apiVersion: ran.openshift.io/v1
kind: SiteConfig
metadata:
  name: "5glab"
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
  - clusterName: "sno2"
    # The sdn plugin that will be used
    networkType: "OVNKubernetes"
    # All Composable capabilities removed except required for telco
    installConfigOverrides:  "{\"capabilities\":{\"baselineCapabilitySet\": \"None\", \"additionalEnabledCapabilities\": [ \"marketplace\", \"NodeTuning\" ] }}"
    extraManifestPath: ./extramanifests
####    extraManifests:
####      filter:
####        inclusionDefault: exclude
####        include:
####          - CS.yaml
    # Cluster labels (this will be used by RHACM)
    clusterLabels:
      common: "ocp414"
      logicalGroup: "active"
      group-du-sno: ""
      du-site: "sno2"
      du-zone: "europe"
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
    additionalNTPSources:
      - infra.5g-deployment.lab
    holdInstallation: false
    nodes:
      - hostName: "sno2.5g-deployment.lab"
        role: "master"
        # We can add custom labels to our nodes, these will be added once the node joins the cluster
        nodeLabels:
          5gran.lab/my-custom-label: ""
        bmcAddress: "redfish-virtualmedia://192.168.125.1:9000/redfish/v1/Systems/local/sno2"
        # The secret name of the secret containing the bmc credentials for our bare metal node
        bmcCredentialsName:
          name: "sno2-bmc-credentials"
        # The MAC Address of the NIC from the bare metal node connected to the machineNetwork
        bootMACAddress: "AA:AA:AA:AA:03:01"
        bootMode: "UEFI"
        rootDeviceHints:
          deviceName: /dev/vda
        nodeNetwork:
          interfaces:
            - name: enp1s0
              macAddress: "AA:AA:AA:AA:03:01"
          config:
            interfaces:
              - name: enp1s0
                type: ethernet
                state: up
                ipv4:
                  enabled: true
                  dhcp: true
                ipv6:
                  enabled: false
EOF
```

Create Kustomization File, pointing to siteconfig.yaml :

```
cat << EOF > ~/ztp/5glab/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
generators:
  - ./siteconfig.yaml
EOF
```

## Create Extra Manifests: 

Extra manifests are beibg created here, but since the siteconfig is not currently pointing to these, the lab exercise demonstrates that these will be ignored: 

```
mkdir -p ~/ztp/5glab/extramanifests
```
```
cat <<EOF > ~/ztp/5glab/extramanifest/CS.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: redhat-operator-index-disconnect
  namespace: openshift-marketplace
spec:
  image: quay.apps.mano-npss.jnpr.bos2.lab/olmidx/olmidx-redhat-operator-index:v4.10
  sourceType: grpc
EOF
``` 
```
cat <<EOF > ~/ztp/5glab/extramanifest/enable-crun-master.yaml
---
apiVersion: machineconfiguration.openshift.io/v1
kind: ContainerRuntimeConfig
metadata:
 name: enable-crun-master
spec:
 machineConfigPoolSelector:
   matchLabels:
     pools.operator.machineconfiguration.openshift.io/master: ""
 containerRuntimeConfig:
   defaultRuntime: crun
EOF
```

## Generate manifests using SiteConfig:

We can now use the ztp plugin to translate the SiteConfig into the target manifests. 

#### Install Kustomize binary and set plugin path:

The plugin is based on kustomize tool. Since we are intending to use it directly (outside of OpenShift environment), we will need to install Kustomize tool locally: 

```
export KUSTOMIZE_PLUGIN_HOME=/root/ztp/plugin
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
mv kustomize /usr/bin/
cd ~/ztp/5glab/
```

#### Generate: 
```
kustomize build ./ --enable-alpha-plugins > out_with_all_extramanifests.yaml
```

Check out the generated manifests: 
```
grep kind\: out_with_all_extramanifests.yaml 
```
Output will look like: 
> kind: Namespace<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: MachineConfig<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: CatalogSource<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: Node<br>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: ContainerRuntimeConfig<br>
> kind: ConfigMap<br>
> kind: InfraEnv<br>
> kind: NMStateConfig<br>
> kind: KlusterletAddonConfig<br>
> kind: ManagedCluster<br>
> kind: AgentClusterInstall<br>
> kind: ClusterDeployment<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: AgentClusterInstall<br>
> kind: BareMetalHost<br>


#### Generate manifets with selective additional Extra-Manifests:

Lets uncomment the lines related to extraManifests. This will achieve the followings: 
* default extraManifests will no longer be used
* The custom defined extraManifest will be included when interpreting the siteconfig file


```
sed -i s/####//g siteconfig.yaml
kustomize build ./ --enable-alpha-plugins > out_with_selective_extramanifests.yaml
```
Check out the generated manifests: 
```
grep kind\: out_with_selective_extramanifests.yaml 
```
Output will look like: 
> kind: Namespace<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: CatalogSource<br>
> kind: ConfigMap<br>
> kind: InfraEnv<br>
> kind: NMStateConfig<br>
> kind: KlusterletAddonConfig<br>
> kind: ManagedCluster<br>
> kind: AgentClusterInstall<br>
> kind: ClusterDeployment<br>
>     kind: AgentClusterInstall<br>
> kind: BareMetalHost<br>


