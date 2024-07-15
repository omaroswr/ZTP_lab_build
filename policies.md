## Lab to explore PGT translation & templating: 

### Create File strcutre: 

Note: the steps defined in siteconfig lab would need to be completed before running this lab exercise. 

```
mkdir -p ~/ztp/5glabpolicies/source-crs
cd ~/ztp/5glabpolicies
```

### Create a PolicyGenTemplate File: 

```
cat << EOF > pgt.yaml
---
apiVersion: ran.openshift.io/v1
kind: PolicyGenTemplate
metadata:
  name: "cwl-site1"
  namespace: "ztp-policy"
spec:
  bindingRules:
    cluster: cwl-site1
  sourceFiles:
    #
    # Label node for Storage
    #
    - fileName: label.yaml
      policyName: "label-nodes"
      metadata:
        name: master1.cwl-site1.npss.bos2.lab
EOF
```

### Create a custom defined source CR: 
```
cat << EOF > source-crs/label.yaml
---
apiVersion: v1
kind: Node
metadata:
  labels:
    cluster.ocs.openshift.io/openshift-storage: "true"
  name: $node
  annotations:
    ran.openshift.io/ztp-deploy-wave: "1"
spec: {}
EOF
```
### Create a kustomization file: 
```
cat << EOF > kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

generators:
- pgt.yaml
EOF
```

### Run the kustomization command with plugin: 

```
export KUSTOMIZE_PLUGIN_HOME=/root/ztp/plugin
kustomize build ./ --enable-alpha-plugins > output.yaml
```

```
grep ^kind\: output.yaml
```
> kind: PlacementRule<br>
> kind: PlacementBinding<br>
> kind: Policy<br>

