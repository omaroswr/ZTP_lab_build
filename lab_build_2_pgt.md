## Configuring Policies:

### Common Policy:

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/common-415.yaml
---
apiVersion: ran.openshift.io/v1
kind: PolicyGenTemplate
metadata:
  name: "common"
  namespace: "ztp-policies"
spec:
  bindingRules:
    common: "ocp415"
    logicalGroup: "active"
  mcp: master
  remediationAction: inform
  sourceFiles:
    - fileName: OperatorHub.yaml
      policyName: config-policies
    - fileName: DefaultCatsrc.yaml
      metadata:
        name: redhat-operator-index
      spec:
        image: infra.5g-deployment.lab:8443/redhat/redhat-operator-index:v4.15-1711601986
      policyName: config-policies
    - fileName: ReduceMonitoringFootprint.yaml
      policyName: config-policies
    - fileName: StorageLVMOSubscriptionNS.yaml
      metadata:
        annotations:
          workload.openshift.io/allowed: management
      policyName: subscription-policies
    - fileName: StorageLVMOSubscriptionOperGroup.yaml
      policyName: subscription-policies
    - fileName: StorageLVMOSubscription.yaml
      spec:
        name: lvms-operator
        channel: stable-4.15
        source: redhat-operator-index
      policyName: subscription-policies
    - fileName: LVMOperatorStatus.yaml
      policyName: subscription-policies
    - fileName: SriovSubscriptionNS.yaml
      policyName: subscription-policies
    - fileName: SriovSubscriptionOperGroup.yaml
      policyName: subscription-policies
    - fileName: SriovSubscription.yaml
      spec:
        channel: stable
        source: redhat-operator-index
      policyName: subscription-policies
    - fileName: SriovOperatorStatus.yaml
      policyName: subscription-policies
EOF
```

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
generators:
- common-415.yaml
EOF
```

### Custom Source-CR example

We will create a custom source-CR that will disable the default catalog sources on the managed cluster. This CR is already being referenced in the PGT that was created above.

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/source-crs/OperatorHub.yaml
apiVersion: config.openshift.io/v1
kind: OperatorHub
metadata:
    name: cluster
    annotations:
        ran.openshift.io/ztp-deploy-wave: "1"
spec:
    disableAllDefaultSources: true
EOF
```

### Namespace for policies & clusterimageset:

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/resources/policies-namespace.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: ztp-policies
  labels:
    name: ztp-policies
EOF
```

The cluster image resource being used by clusters is the (disconnected) registry that is hosted locally on this bastion. The `siteconfig` files refer to this resource by using `clusterImageSetNameRef: "active-ocp-version"`.  So this needs to be defined as well. As this is relevant to policy, its manifest is being created under the `policy/resources` location: 


```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/resources/cluster-image-set.yaml
---
apiVersion: hive.openshift.io/v1
kind: ClusterImageSet
metadata:
  name: active-ocp-version
spec:
  releaseImage: infra.5g-deployment.lab:8443/openshift/release-images:4.15.0-x86_64
EOF
```

Kustomization for the namespace: 

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/resources/kustomization.yaml
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - policies-namespace.yaml
  - cluster-image-set.yaml
EOF
```

### Top level kustomization : 

```
cat <<EOF > ~/5g-deployment-lab/ztp-repository/policies/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- resources
#- site-specific-policies
#- configuration-version-2023-09-10_802e68c
#- configuration-version-2023-11-01_39ba72d
- configuration-version-2024-03-04
EOF
```

### Final View: 

The directory structure at this point should look like the following: 

```
tree ~/5g-deployment-lab/
```

![image4](images/lab_build_2.png)

### Commit to GIT: 

```
cd ~/5g-deployment-lab/ztp-repository
git add --all
git commit -m 'Added policies information'
git push origin main
cd ~
```
