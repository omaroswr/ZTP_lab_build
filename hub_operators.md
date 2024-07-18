## Disable default catalog sources: 

```
cat << EOF | oc apply -f -
apiVersion: config.openshift.io/v1
kind: OperatorHub
metadata:
  name: cluster
spec: 
  disableAllDefaultSources: true
```

## add new catalog source: 

```
cat ~/oc-mirror-workspace/$(ls ~/oc-mirror-workspace/ | grep result)/catalog*.yaml | oc apply -f -
```

Check that the new catalog source is available using: 

`oc get catalogsource -A`

## Check which operators are available using the new catalog source:

Try the following: 
```
oc get packagemanifests.packages.operators.coreos.com
NAME                               CATALOG   AGE
multicluster-engine                          89s
quay-operator                                89s
local-storage-operator                       89s
topology-aware-lifecycle-manager             89s
cluster-logging                              89s
advanced-cluster-management                  89s
openshift-gitops-operator                    89s
```

## Check the catalog source for version information: 
Use the follwing to find out the version available:
```
oc get packagemanifest advanced-cluster-management -o=jsonpath='{.status.channels[*].name}{"\n"}'
```
Output will show you:

> release-2.10 release-2.9

## ACM:

```
apiVersion: project.openshift.io/v1
kind: Project
metadata:
  name: open-cluster-management
  labels:
    kubernetes.io/metadata.name: open-cluster-management
spec: {}
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: acm
  namespace: open-cluster-management
spec:
  targetNamespaces:
  - open-cluster-management
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: acm-operator-subscription
  namespace: open-cluster-management
spec:
  sourceNamespace: openshift-marketplace
  source: cs-redhat-operator-index
  channel: release-2.9
  installPlanApproval: Automatic
  name: advanced-cluster-management
```

## MCE:

(See: https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.5/html/install/installing#installing-in-a-disconnected-environment)
```
apiVersion: operator.open-cluster-management.io/v1
kind: MultiClusterHub
metadata:
  name: multiclusterhub
  namespace: open-cluster-management
  annotations:
    installer.open-cluster-management.io/mce-subscription-spec: '{"source": "cs-redhat-operator-index"}'
spec: {}
```


## Logging: 

```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  targetNamespaces:
  - openshift-logging
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  channel: "stable-5.9"
  name: cluster-logging
  source: cs-redhat-operator-index
  sourceNamespace: openshift-marketplace
---
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogging"
metadata:
  name: "instance"
  namespace: "openshift-logging"   
spec:
  managementState: "Managed"
  collection:
    logs:
      type: "vector"
      fluentd: {}
```

## TALM: 

```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-topology-aware-lifecycle-manager-subscription
  namespace: openshift-operators
spec:
  channel: "stable"
  name: topology-aware-lifecycle-manager
  source: cs-redhat-operator-index
  sourceNamespace: openshift-marketplace
```

## GITOPS
```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-operators
spec:
  channel: latest 
  installPlanApproval: Automatic
  name: openshift-gitops-operator 
  source: cs-redhat-operator-index 
  sourceNamespace: openshift-marketplace 
```


For VNC: https://www.ibm.com/support/pages/how-configure-vnc-server-red-hat-enterprise-linux-8
For releease info: `oc adm release info --insecure=true --idms-file=/root/oc-mirror-workspace/results-1714073520/imageContentSourcePolicy.yaml`

