
### First lets add the Helm Chart Repository
Run `helm repo add starrocks-community https://starrocks.github.io/helm-charts`{{exec}}

### Then make sure that we have the latest charts
Run `helm repo update`{{exec}}

### Can't hurt to double check the versions of the chart
Run `helm search repo starrocks-community`{{exec}}

### Now we can do the standard install.  This will give you one Front End Node (FE) and a Back End Node (BE)
Run `helm install starrocks starrocks-community/kube-starrocks`{{exec}}

### Let's check what pods where installed
Run `kubectl get pods`{{exec}}

### Finally check on the status of the deployment
Run `kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"`{{exec}}


