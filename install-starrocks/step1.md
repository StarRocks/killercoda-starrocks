
Run `helm repo add starrocks-community https://starrocks.github.io/helm-charts`{{exec}}
Run `helm repo update`{{exec}}
Run `helm search repo starrocks-community`{{exec}}
Run `helm install starrocks starrocks-community/kube-starrocks`{{exec}}
Run `kubectl get pods`{{exec}}
Run `kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"`{{exec}}


