# install metrics

https://github.com/kubernetes/kops/tree/master/addons/metrics-server

``` cmd
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/metrics-server/v1.8.x.yaml
```

PS: replace `k8s.gcr.io` with `registry.aliyuncs.com/google_containers`

``` cmd
helm upgrade --install metrics stable/metrics-server --namespace kube-system
```
