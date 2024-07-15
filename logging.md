```
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
EOF
```


```
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  targetNamespaces:
  - openshift-logging
EOF
```

Find the channel names avaialable: 

```
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  channel: "stable"
  name: cluster-logging
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

Check the CSV:
```
oc get csv -n openshift-logging 
```
> NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DISPLAY&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VERSION&nbsp;&nbsp;&nbsp;REPLACES&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PHASE<br>
> cluster-logging.v5.8.3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Red&nbsp;Hat&nbsp;OpenShift&nbsp;Logging&nbsp;&nbsp;&nbsp;5.8.3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cluster-logging.v5.8.2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Succeeded<br>
> openshift-gitops-operator.v1.11.1&nbsp;&nbsp;&nbsp;Red&nbsp;Hat&nbsp;OpenShift&nbsp;GitOps&nbsp;&nbsp;&nbsp;&nbsp;1.11.1&nbsp;&nbsp;&nbsp;&nbsp;openshift-gitops-operator.v1.11.0&nbsp;&nbsp;&nbsp;Succeeded<br>

```
cat << EOF | oc apply -f -
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
EOF
```

```
oc get clusterloggings.logging.openshift.io -A
```
> NAMESPACE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MANAGEMENT&nbsp;STATE<br>
> openshift-logging&nbsp;&nbsp;&nbsp;instance&nbsp;&nbsp;&nbsp;Managed<br>

