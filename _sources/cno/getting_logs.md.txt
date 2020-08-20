# Getting logs
1. You create a debug pod with a fedora image on that node `oc debug --image fedora:31 node/<node name>`
2. In a separate terminal session you do: `oc rsync $DEBUG_POD:/host/$LOCATION_ON_NODE $LOCATION_ON_YOUR_LOCAL_MACHINE`
