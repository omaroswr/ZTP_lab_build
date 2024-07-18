For the steps in this lab, assumr root role by using:

```
sudo -i
```

### Verify BMC Connectivity:
```
curl -k https://127.0.0.1:9000/redfish/v1/Systems/local/sno2
```
> {<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"@odata.type":&nbsp;"#ComputerSystem.v1_1_0.ComputerSystem",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"Id":&nbsp;"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaa0301",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"Name":&nbsp;"sno2",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"UUID":&nbsp;"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaa0301",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"Manufacturer":&nbsp;"kvm",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"Status":&nbsp;{<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"State":&nbsp;"Enabled",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Health":&nbsp;"OK",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"HealthRollUp":&nbsp;"OK"<br>
> &nbsp;&nbsp;&nbsp;&nbsp;},<br>
> ....snip.....<br>

### Verify DNS: 
```
nslookup api.sno2.5g-deployment.lab
```
> Server:&nbsp;&nbsp;192.168.125.1<br>
> Address:&nbsp;192.168.125.1#53<br>
> <br>
> Name:&nbsp;api.sno2.5g-deployment.lab<br>
> Address: 192.168.125.40<br>

```
nslookup console-openshift-console.apps.sno2.5g-deployment.lab
```
> Server:&nbsp;&nbsp;192.168.125.1<br>
> Address:&nbsp;192.168.125.1#53<br>
> <br>
> Name:&nbsp;console-openshift-console.apps.sno2.5g-deployment.lab<br>
> Address: 192.168.125.40<br>

### Verify GIT Access: 
Point your browsser to: `http://infra.5g-deployment.lab:3000/` 

You should see the Gitea GUI 

Try cloning from this GIT Repo: 

```
git clone http://infra.5g-deployment.lab:3000/student/5g-ran-deployments-on-ocp-lab.git
```
> Cloning&nbsp;into&nbsp;'5g-ran-deployments-on-ocp-lab'...<br>
> remote:&nbsp;Enumerating&nbsp;objects:&nbsp;2934,&nbsp;done.<br>
> remote:&nbsp;Counting&nbsp;objects:&nbsp;100%&nbsp;(2934/2934),&nbsp;done.<br>
> remote:&nbsp;Compressing&nbsp;objects:&nbsp;100%&nbsp;(1019/1019),&nbsp;done.<br>
> remote:&nbsp;Total&nbsp;2934&nbsp;(delta&nbsp;1727),&nbsp;reused&nbsp;2890&nbsp;(delta&nbsp;1693),&nbsp;pack-reused&nbsp;0<br>
> Receiving&nbsp;objects:&nbsp;100%&nbsp;(2934/2934),&nbsp;17.86&nbsp;MiB&nbsp;|&nbsp;82.76&nbsp;MiB/s,&nbsp;done.<br>
> Resolving&nbsp;deltas:&nbsp;100%&nbsp;(1727/1727),&nbsp;done.<br>


### Verify Registry Available and Populated:
```
podman login --authfile auth.json -u admin  infra.5g-deployment.lab:8443 -p r3dh4t1!
```
> Login Succeeded!

```
 podman exec -it registry ls /registry/docker/registry/v2/repositories/
```
> ansible-automation-platform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;multicluster-engine&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;openshift4&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rhel8<br>
> ansible-automation-platform-24&nbsp;&nbsp;oc-mirror&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;redhat&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rhsysdeseng<br>
> karmab&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;openshift&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rh-sso-7&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ubi8<br>
> lvms4&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;openshift-gitops-1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rhacm2<br>

### Verify Operators:

```
oc get operators
```
> NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AGE<br>
> advanced-cluster-management.open-cluster-management&nbsp;&nbsp;&nbsp;&nbsp;132m<br>
> ansible-automation-platform-operator.aap&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;132m<br>
> lvms-operator.openshift-storage&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;135m<br>
> multicluster-engine.multicluster-engine&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;132m<br>
> openshift-gitops-operator.openshift-operators&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;135m<br>
> topology-aware-lifecycle-manager.openshift-operators&nbsp;&nbsp;&nbsp;132m<br>


```
oc get csv -n open-cluster-management
```
> NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DISPLAY&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VERSION&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;REPLACES&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PHASE<br>
> aap-operator.v2.4.0-0.1698896316&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ansible&nbsp;Automation&nbsp;Platform&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.4.0+0.1698896316&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Succeeded<br>
> advanced-cluster-management.v2.9.0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Advanced&nbsp;Cluster&nbsp;Management&nbsp;for&nbsp;Kubernetes&nbsp;&nbsp;&nbsp;2.9.0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Succeeded<br>
> openshift-gitops-operator.v1.10.1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Red&nbsp;Hat&nbsp;OpenShift&nbsp;GitOps&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.10.1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;openshift-gitops-operator.v1.10.0&nbsp;&nbsp;&nbsp;Succeeded<br>
> topology-aware-lifecycle-manager.v4.14.0&nbsp;&nbsp;&nbsp;Topology&nbsp;Aware&nbsp;Lifecycle&nbsp;Manager&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.14.0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Succeeded<br>

### Verify ZTP Plugin: 

```
 oc -n openshift-gitops get argocd openshift-gitops -o jsonpath='{.spec.repo.initContainers[0].image}{"\n"}'
```
> infra.5g-deployment.lab:8443/openshift4/ztp-site-generate-rhel8:v4.15.0-32


### Verify AgentServiceConfig: 

```
oc get agentserviceconfig agent -o jsonpath={.spec.osImages} | jq
```

> [<br>
> &nbsp;&nbsp;{<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"cpuArchitecture":&nbsp;"x86_64",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"openshiftVersion":&nbsp;"4.15",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"rootFSUrl":&nbsp;"http://infra.5g-deployment.lab:8080/rhcos-4.15.0-x86_64-live-rootfs.x86_64.img",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"url":&nbsp;"http://infra.5g-deployment.lab:8080/rhcos-4.15.0-x86_64-live.x86_64.iso",<br>
> &nbsp;&nbsp;&nbsp;&nbsp;"version":&nbsp;"415.92.202402201450-0"<br>
> &nbsp;&nbsp;}<br>
