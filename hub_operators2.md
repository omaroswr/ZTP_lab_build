
## Disable default catalog sources: 

```
cat << EOF | oc apply -f -
apiVersion: config.openshift.io/v1
kind: OperatorHub
metadata:
  name: cluster
spec: 
  disableAllDefaultSources: true
EOF
```

## add new catalog source: 

```
for cs_filename in ~/oc-mirror-workspace/result*/catalog*.yaml; do oc apply -f $cs_filename; done
cp ~/oc-mirror-workspace/result*/*redhat*.yaml ~/new.yaml
sed -i s/cs-redhat-operator-index/redhat-operators/g new.yaml
oc apply -f ~/new.yaml
```

Check that the new catalog source is available using: 

```
oc get catalogsource -A
```

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
patterns-operator                            89s
```

## Complete validated patterns subscription:

Validated Pattern operator subscription was already included in the installation iso file
However, the pattern will not install yet since it doesn't support disconneced environment.  To work around that, we can tempoararily enable proxy configuration on our cluster. 

Prior to do that, lets install proxy server on the host. Exit from bastion (sing "CTRL+]") console, and use the following command at the `[root@hypervisor ~]#` prompt:

```
dnf install -y squid
systemctl start squid
systemctl enable squid
```
While on the host system, note down the IP address it uses, using `ip addr show | grep "bond0$"` . For example: 
```
ip addr show | grep "bond0$"
    inet 147.28.185.223/31 scope global noprefixroute bond0
```

Reconnect to basion using : `virsh console bastion` OR `ssh root@192.168.125.10`

First add all the credentials to the pull secret. The original cluster was built with a pull-secret that only had credentials for the local Quay. Now since we are enabling proxy, we need to add the full pull-secret to the cluster. Run the following commands on the bastion:

```
AUTHSTRING="{\"quay.tnc.bootcamp.lab:8443\": {\"auth\": \"cXVheTpzeWVkQHJlZGhhdA==\",\"email\": \"quay@redhat.com\"}}"

jq -c ".auths += $AUTHSTRING" < /root/.docker/config.json > /tmp/pull-secret.json

oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=/tmp/pull-secret.json
```

Now configure the proxy CR on the cluster to use this proxy. For that, first create a yaml file called `proxy.yaml` using the following sample: 
```
cat << EOF > proxy.yaml
apiVersion: config.openshift.io/v1
kind: Proxy
metadata:
  name: cluster
spec:
  httpProxy: http://147.28.185.223:3128
  httpsProxy: http://147.28.185.223:3128
  noProxy: .cluster.local,.svc,10.128.0.0/14,127.0.0.1,172.30.0.0/16,192.168.125.0/24,192.168.126.0/24,api-int.hub.tnc.bootcamp.lab,fd02::/48,localhost,tnc.bootcamp.lab,quay.tnc.bootcamp.lab,api.hub.tnc.bootcamp.lab,git.tnc.bootcamp.lab,8443,3000,6443
EOF
```
here, the IP address for `httpProxy` and `httpsProxy` should be one you had extracted for your own cluster. 

Then apply the file using:
```
oc apply -f proxy.yaml
```

The cluster will go through a rolling update to apply the proxy change. The SNO node will reboot for the change to take effect.

Once the node is ready again, Verify if the patterns operator is properly installed, using `oc get csv` , for example: 

```
oc get csv
NAME                        DISPLAY                       VERSION   REPLACES                    PHASE
patterns-operator.v0.0.52   Validated Patterns Operator   0.0.52    patterns-operator.v0.0.51   Succeeded
```

Note, if the CSV phase shows as "Failed" , then you can remove and readd it by using: 
```
oc delete -f ~/vpattern.yaml
oc delete csv -n openshift-operators patterns-operator.v0.0.52
oc delete operators patterns-operator.openshift-operators
sleep 10
oc apply -f ~/vpattern.yaml
```

The CSV should get recreated and reach "Succeed" state

## Using the validated patterns operator: 


To start using the validated pattern operator, lets first create a namespace for it to use: 

```
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
  name: mgmt-gitops-hub
EOF
```

Now, create secret CR that will be used by the operator's instance to login to Git using the provided credentials: 

```
cat << EOF | oc apply -f -
kind: Secret
apiVersion: v1
metadata:
  name: private-git-repo-creds
  namespace: mgmt-gitops-hub
  labels:
    argocd.argoproj.io/secret-type: repository
data:
  password: aGFzc2Fu
  type: Z2l0
  url: aHR0cDovL2dpdC50bmMuYm9vdGNhbXAubGFiOjMwMDAvc3llZC92cGF0dGVybi5naXQ=
  username: c3llZA==
type: Opaque
EOF
```

Finally, now create the Pattern CR, which will enable this operator to start using Git as the source for the content it should install on the clsuter: 


```
cat << EOF | oc apply -f - 
apiVersion: gitops.hybrid-cloud-patterns.io/v1alpha1
kind: Pattern
metadata:
  labels:
    app.kubernetes.io/managed-by: Helm
  name: mgmt-gitops
  namespace: openshift-operators
spec:
  clusterGroupName: hub
  gitOpsSpec:
    manualSync: true
    operatorChannel: latest
    operatorSource: cs-redhat-operator-index
  gitSpec:
    targetRepo: http://git.tnc.bootcamp.lab:3000/syed/vpattern.git
    targetRevision: main
    tokenSecret: private-git-repo-creds
    tokenSecretNamespace: mgmt-gitops-hub
EOF
```

This will result in installation of ArgoCD (GitOps) operator. The following verifies that: 

```
oc get csv
NAME                                DISPLAY                       VERSION   REPLACES                            PHASE
openshift-gitops-operator.v1.12.4   Red Hat OpenShift GitOps      1.12.4    openshift-gitops-operator.v1.12.3   Succeeded
patterns-operator.v0.0.52           Validated Patterns Operator   0.0.52    patterns-operator.v0.0.51           Succeeded
```
## Start ArgoCD Sync

Normally the sync starts automatically, but in this lab environment that is not happening. So we will manually trigger the sync.

Figure out the argoCD route:

```
oc -n openshift-gitops get route
```

Figure out the ArgoCD admin password:

```
oc -n openshift-gitops extract secret/openshift-gitops-cluster --to=-
```

Point your browser to the argoCD route and login as admin using the credentials returned from the previous command. Go to the application and start the sync.

## Check App Sync Status


vPattern operator created the argoCD instance in `openshift-gitops` namespace. That argoCD instance synced with GIT and created an application:

```
oc -n openshift-gitops get apps
```

This application installs the operators that are defined in the `values-hub.yaml` file of the vpattern.git repository, as well as creates several applications in the `mgmt-gitops-hub` namespace to configure those operators:

```
oc -n mgmt-gitops-hub get apps
```

A new instance of argoCD is also created in the `mgmt-gitops-hub` namespace. Connect to the GUI of that argoCD instance and check the status of each application.


<!--

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
-->
