```
sed -i "s/  source\:/  ignoreDifferences\:\n  - group\: \"extensions.hive.openshift.io\"\n    kind\: \AgentClusterInstall\"\n    jsonPointers\:\n    - \/spec\/provisionRequirements\n  source\:/g" ~/5g-deployment-lab/deployment/cluster-app.yaml 
```
```
sed -i "s/    syncOptions\:/    syncOptions\:\n    - RespectIgnoreDifferences\=true/g" ~/5g-deployment-lab/deployment/cluster-app.yaml
```
```
oc apply -f ~/5g-deployment-lab/deployment/cluster-app.yaml
```

```
kcli show vm sno2w | grep mac
```
> net interface: eth0 mac: aa:aa:aa:aa:04:01 net: 5gdeploymentlab type: routed

```
cat << EOF >> ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/5glab.yaml
      - hostName: "sno2w.5g-deployment.lab"
        role: "worker"
        # We can add custom labels to our nodes, these will be added once the node joins the cluster
        nodeLabels:
          5gran.lab/my-worker: ""
        bmcAddress: "redfish-virtualmedia://192.168.125.1:9000/redfish/v1/Systems/local/sno2w"
        # The secret name of the secret containing the bmc credentials for our bare metal node
        bmcCredentialsName:
          name: "sno2w-bmc-credentials"
        # The MAC Address of the NIC from the bare metal node connected to the machineNetwork
        bootMACAddress: "AA:AA:AA:AA:04:01"
        bootMode: "UEFI"
        rootDeviceHints:
          deviceName: /dev/vda
        nodeNetwork:
          interfaces:
            - name: enp1s0
              macAddress: "AA:AA:AA:AA:04:01"
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

```
cat << EOF >> ~/5g-deployment-lab/ztp-repository/clusters/site-group-1/secrets/sno2/bmc-credentials.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: sno2w-bmc-credentials
  namespace: sno2
data:
  username: "YWRtaW4="
  password: "YWRtaW4="
type: Opaque
EOF
```

```
cd ~/5g-deployment-lab/ztp-repository/
git add .
git commit -m "adding sno2 worker"
git push
```

```
while true; do echo "############## echo -n `date` ##############"; echo "---------------- BMH ----------------"; oc get bmh -A; echo "---------------- AgentClusterInstall ----------------"; oc get agentclusterinstall -A; echo "---------------- SNO2 Cluster ----------------"; oc --kubeconfig=/root/sno2-kubeconfig get nodes; sleep 15; done
```

```
oc get nodes --kubeconfig ~/sno2-kubeconfig 
```
> NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;STATUS&nbsp;&nbsp;&nbsp;ROLES&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VERSION<br>
> sno2.5g-deployment.lab&nbsp;&nbsp;&nbsp;&nbsp;Ready&nbsp;&nbsp;&nbsp;&nbsp;control-plane,master,worker&nbsp;&nbsp;&nbsp;5h57m&nbsp;&nbsp;&nbsp;v1.27.6+f67aeb3<br>
> sno2w.5g-deployment.lab&nbsp;&nbsp;&nbsp;Ready&nbsp;&nbsp;&nbsp;&nbsp;worker&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;69m&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v1.27.6+f67aeb3<br>

