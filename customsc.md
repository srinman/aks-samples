# Custom StorageClass  

Custom storageclass is recommended to create storage objects based on unique requirements.  For example, you may need to use a different resource group to keep storage accounts, use your own VNET, subnet. 

https://learn.microsoft.com/en-us/azure/aks/azure-files-csi?source=recommendations#create-a-custom-storage-class  


Following sample creates Standard_LRS type storage in myappstoragerg resource group and it creates private endpoint in pesubt which is in aks-vnet aksresourcerg.   You can create more custom storage classes with different parameters based on requirements.    

Cluster managed identity needs to be provided appropriate access (if placed in different resource group)

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-azfile-lrs-myapp
provisioner: file.csi.azure.com
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - dir_mode=0640
  - file_mode=0640
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict # https://linux.die.net/man/8/mount.cifs
  - nosharesock
parameters:
  skuName: Standard_LRS
  resourceGroup: myappstoragerg
  vnetResourceGroup: aksresourcerg
  vnetName: aks-vnet
  subnetName: pesubnet
  networkEndpointType: privateEndpoint
```

PVC
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-pvc1
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: sc-azfile-lrs-myapp
  resources:
    requests:
      storage: 10Gi
```