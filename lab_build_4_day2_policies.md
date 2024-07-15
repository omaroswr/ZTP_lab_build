## Create new manifests to be used by day2 policy: 

While exploring day-2 policies, lets make use of the ability to create custom manifests as well. The following will be creatged and placed in source-crsdirectory. 


```
mkdir ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/source-crs/LABEL
```

```
cat << EOF > ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/source-crs/LABEL/LabelNodeOCS.yaml
---
apiVersion: v1
kind: Node
metadata:
  labels:
    cluster.ocs.openshift.io/openshift-storage: "true"
  name: $node
spec: {}
EOF
```

Note that the manifests are using two variables, which will need to be specified when using these CRs in the PGT manifests

### Creating PGT to be applied: 

```
cat << EOF > ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/sno2-label.yaml
apiVersion: ran.openshift.io/v1
kind: PolicyGenTemplate
metadata:
  name: "sno2"
  namespace: "ztp-policies"
spec:
  bindingRules: 
    du-site: "sno2"
    logicalGroup: "active"
  mcp: master 
  remediationAction: inform
  sourceFiles:
    - fileName: LABEL/LabelNodeOCS.yaml
      policyName: "label-nodes"
      metadata: 
        name: sno2.5g-deployment.lab
EOF
```

### Add the new policy to kustomization: 
```
echo "- sno2-label.yaml" >> ~/5g-deployment-lab/ztp-repository/policies/configuration-version-2024-03-04/kustomization.yaml 
```

### Push poicy to Git:

```
cd ~/5g-deployment-lab/ztp-repository/
git add .
git commit -m "adding day2 policy for sno2" 
git push
```

### Verify that policies are rendered:
```
oc get policy -A | grep label
```
> sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ztp-policies.sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NonCompliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;21s<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NonCompliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;21s<br>

**NOTE** It might take a little time for GitOps to sync with the latest change in Git, and render the above pokicies You can expedite that by using the REFRESH option in the ArgoCD GUI for the policies App. 

### Create CGU:

Before we apply the policy by create CGU, lets first see the state of node's labels for SNO2: 

```
oc get nodes --show-labels --kubeconfig ~/sno2-kubeconfig 
```
> NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;STATUS&nbsp;&nbsp;&nbsp;ROLES&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AGE&nbsp;&nbsp;&nbsp;VERSION&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LABELS<br><br>
> sno2.5g-deployment.lab&nbsp;&nbsp;&nbsp;Ready&nbsp;&nbsp;&nbsp;&nbsp;control-plane,master,worker&nbsp;&nbsp;&nbsp;79m&nbsp;&nbsp;&nbsp;v1.27.6+f67aeb3&nbsp;&nbsp;&nbsp;5gran.lab/my-custom-label=,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=sno2.5g-deployment.lab,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node-role.kubernetes.io/worker=,node.openshift.io/os_id=rhcos<br>           
> <br>

Also check status of currenct CGUs: 

```
oc get cgu -A
```

> NAMESPACE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;STATE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DETAILS<br>
> ztp-install&nbsp;&nbsp;&nbsp;&nbsp;local-cluster&nbsp;&nbsp;&nbsp;3h22m&nbsp;&nbsp;&nbsp;Completed&nbsp;&nbsp;&nbsp;&nbsp;All&nbsp;clusters&nbsp;already&nbsp;compliant&nbsp;with&nbsp;the&nbsp;specified&nbsp;managed&nbsp;policies<br>
> ztp-install&nbsp;&nbsp;&nbsp;&nbsp;sno1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;59m&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Completed&nbsp;&nbsp;&nbsp;&nbsp;All&nbsp;clusters&nbsp;already&nbsp;compliant&nbsp;with&nbsp;the&nbsp;specified&nbsp;managed&nbsp;policies<br>


Now you can apply the CGU, which will trigger the policy enforcement: 

```
cat << EOF | oc apply -f -
apiVersion: ran.openshift.io/v1alpha1
kind: ClusterGroupUpgrade
metadata:
  name: sno2-day2
  namespace: ztp-policies
spec:
  clusters:
  - sno2
  enable: true
  managedPolicies:
  - sno2-label-nodes
  preCaching: false
  remediationStrategy:
    maxConcurrency: 1
    timeout: 240
EOF
```

Now check if the CGU is applied: 

```
oc get cgu -A | grep sno2
```

> ztp-install&nbsp;&nbsp;&nbsp;&nbsp;sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;67m&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Completed&nbsp;&nbsp;&nbsp;&nbsp;All&nbsp;clusters&nbsp;are&nbsp;compliant&nbsp;with&nbsp;all&nbsp;the&nbsp;managed&nbsp;policies<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-day2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4s&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InProgress&nbsp;&nbsp;&nbsp;Remediating&nbsp;non-compliant&nbsp;policies<br>

Check status of policies: 

```
oc get policy -A | grep label
```
> sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ztp-policies.sno2-day2-sno2-label-nodes-68ffc&nbsp;&nbsp;&nbsp;&nbsp;enforce&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;13s<br>
> sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ztp-policies.sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NonCompliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11m<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-day2-sno2-label-nodes-68ffc&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;enforce&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;13s<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NonCompliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11m<br>
> <br>

After few minutes, the policies will reach a compliant status: 
```
oc get policy -A | grep label
```

> sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ztp-policies.sno2-day2-sno2-label-nodes-68ffc&nbsp;&nbsp;&nbsp;&nbsp;enforce&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;33s<br>
> sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ztp-policies.sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11m<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-day2-sno2-label-nodes-68ffc&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;enforce&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;33s<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11m<br>
~                                                                                      

At this point, you can check the node's label, and see the new label is now applied to the node as expected:

```
oc get nodes --show-labels --kubeconfig ~/sno2-kubeconfig 
```

> NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;STATUS&nbsp;&nbsp;&nbsp;ROLES&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AGE&nbsp;&nbsp;&nbsp;VERSION&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LABELS<br>
> sno2.5g-deployment.lab&nbsp;&nbsp;&nbsp;Ready&nbsp;&nbsp;&nbsp;&nbsp;control-plane,master,worker&nbsp;&nbsp;&nbsp;82m&nbsp;&nbsp;&nbsp;v1.27.6+f67aeb3&nbsp;&nbsp;&nbsp;5gran.lab/my-custom-label=,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,**cluster.ocs.openshift.io/openshift-storage=true**,kubernetes.io/arch=amd64,kubernetes.io/hostname=sno2.5g-deployment.lab,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node-role.kubernetes.io/worker=,node.openshift.io/os_id=rhcos<br>
> <br>
> <br>
>

After few minutes, the enforced policies will also get deleted:

> sno2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ztp-policies.sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;16m<br>
> ztp-policies&nbsp;&nbsp;&nbsp;sno2-label-nodes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Compliant&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;16m<br>
> <br>

### Verify status of CGU after successful compliance: 

```
oc get cgu -A | grep day2
```
>ztp-policies&nbsp;&nbsp;&nbsp;sno2-day2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;40m&nbsp;&nbsp;&nbsp;&nbsp;Completed&nbsp;&nbsp;&nbsp;All&nbsp;clusters&nbsp;are&nbsp;compliant&nbsp;with&nbsp;all&nbsp;the&nbsp;managed&nbsp;policies<br>
