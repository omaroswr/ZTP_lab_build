```
sudo -i 
#
dnf install -y tree jq bind-utils
oc completion bash > oc_bash_completion
source oc_bash_completion 
mv oc_bash_completion /etc/bash_completion.d/

# kcli list vm

echo "dhcp-host=aa:aa:aa:aa:04:01,ocp-sno2w,192.168.125.41" > /opt/dnsmasq/include.d/sno2w.ipv4

# If adding new node: 
# cp /opt/dnsmasq/include.d/sno2.ipv4 /opt/dnsmasq/include.d/sno3.ipv4
# sed -i s/sno2/sno3/g /opt/dnsmasq/include.d/sno3.ipv4
# sed -i s/40/41/g /opt/dnsmasq/include.d/sno3.ipv4

systemctl restart dnsmasq-virt
kcli create vm -P start=False -P uefi_legacy=true -P plan=hub -P memory=24000 -P numcpus=12 -P disks=[200,200] -P nets=['{"name": "5gdeploymentlab", "mac": "aa:aa:aa:aa:04:01"}'] -P uuid=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaa0401 -P name=sno2w

## kcli list vm
## +----------------+--------+----------------+----------------------------------------------------+------+---------+
## |      Name      | Status |       Ip       |                       Source                       | Plan | Profile |
## +----------------+--------+----------------+----------------------------------------------------+------+---------+
## | hub-ctlplane-0 |   up   | 192.168.125.20 | rhcos-414.92.202310170514-0-openstack.x86_64.qcow2 | hub  |  kvirt  |
## | hub-ctlplane-1 |   up   | 192.168.125.21 | rhcos-414.92.202310170514-0-openstack.x86_64.qcow2 | hub  |  kvirt  |
## | hub-ctlplane-2 |   up   | 192.168.125.22 | rhcos-414.92.202310170514-0-openstack.x86_64.qcow2 | hub  |  kvirt  |
## |      sno1      |   up   | 192.168.125.30 |                                                    | hub  |  kvirt  |
## |      sno2      |  down  |                |                                                    | hub  |  kvirt  |
## |      sno2w     |  down  |                |                                                    | hub  |  kvirt  |
## +----------------+--------+----------------+----------------------------------------------------+------+---------+

## kcli list plan
## +------+-------------------------------------------------------------+
## | Plan |                             Vms                             |
## +------+-------------------------------------------------------------+
## | hub  | hub-ctlplane-0,hub-ctlplane-1,hub-ctlplane-2,sno1,sno2,sno2w|
## +------+-------------------------------------------------------------+
```

Since we are using private IP addressing in the lab, we need to enable SSH tunneling on our laptops, in order to be able to point the browser to the different endpoints/applications used in the lab. Use whatever port forwarding / tunneling tool/mechanism that you are familiar with. On a MAC, the following works:

NOTE: Replace the IP Address in the `HostName` below with the IP address of your RHDP lab.

```
cat ~/.ssh/config
<snip>
Host rhdp
HostName 147.75.203.15
User  lab-user
ServerAliveInterval 300
ServerAliveCountMax 2
```

Enable SSH tunneling for the subnets 192.168.125.0/24 and 192.168.126.0/24:
```
sshuttle -r rhdp 192.168.125.0/24 192.168.126.0/24
```

We also need to add the following  entries in the `/etc/hosts' file on our laptop:

```
# RHDP 5G RAN Lab
192.168.125.1 infra.5g-deployment.lab
192.168.125.10 api.hub.5g-deployment.lab
192.168.125.11 console-openshift-console.apps.hub.5g-deployment.lab
192.168.125.11 oauth-openshift.apps.hub.5g-deployment.lab
192.168.125.11 openshift-gitops-server-openshift-gitops.apps.hub.5g-deployment.lab
192.168.125.40 api.sno2.5g-deployment.lab
192.168.125.40 console-openshift-console.apps.sno2.5g-deployment.lab
```

