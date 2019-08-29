# issue int.parcelfuture.com ssl cert

## install acme.sh

``` shell
wget -O -  https://get.acme.sh | sh
./acme.sh --install
```

## issue cert

``` shell
export DP_Id="ID数字"
export DP_Key="Token字符串"
acme.sh --issue -d *.int.parcelfuture.com -d int.parcelfuture.com --dns dns_dp
```

## trans cert file to k8s format

``` shell
cd /home/azureuser/.acme.sh/*.int.parcelfuture.com
cat fullchain.cer | base64 | xargs echo -n | sed s/[[:space:]]//g> tls.crt
cat *.int.parcelfuture.com.key | base64 | xargs echo -n |sed s/[[:space:]]//g > tls.key
```

## create secret yaml

``` yaml
apiVersion: v1
data:
  ca.crt: ""
  tls.crt: tls.crt
  tls.key: tls.key
kind: Secret
metadata:
  annotations:
    certmanager.k8s.io/alt-names: int.parcelfuture.com
    certmanager.k8s.io/common-name: int.parcelfuture.com
    certmanager.k8s.io/ip-sans: ""
  labels:
    certmanager.k8s.io/certificate-name: int-parcelfuture-com
  name: int-parcelfuture-com
  # namespace: registry
type: kubernetes.io/tls
```

## install secret to k8s

``` shell
kubectl apply -f cert.yaml -n registry
kubectl apply -f cert.yaml -n test
```
