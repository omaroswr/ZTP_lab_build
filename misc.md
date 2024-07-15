```
while true; do echo -n "========="; echo -n `date` ; echo "=========";  echo "========= managed clusters ========"; oc get managedclusters; echo "========= policy ========"; oc get policy -A; echo "========= CGU  ========"; oc get cgu -A; sleep 15; done
```

```
oc get packagemanifest advanced-cluster-management -o=jsonpath='{.status.channels[*].name}{"\n"}'
```
