# create k8s user and set permission

## reference

https://kubernetes.io/docs/reference/access-authn-authz/rbac/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

https://github.com/kubernetes/dashboard/wiki/Creating-sample-user

## Create Service Account

We are creating Service Account with name admin-user in namespace kube-system first.

    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: admin-user
    namespace: kube-system

## Create ClusterRoleBinding

In most cases after provisioning our cluster using kops or kubeadm or any other popular tool, the ClusterRole admin-Role already exists in the cluster. We can use it and create only ClusterRoleBinding for our ServiceAccount.

NOTE: apiVersion of ClusterRoleBinding resource may differ between Kubernetes versions. Prior to Kubernetes v1.8 the apiVersion was rbac.authorization.k8s.io/v1beta1.

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    name: admin-user
    roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
    subjects:
    - kind: ServiceAccount
    name: admin-user
    namespace: kube-system

### Resources list

pods

secrets

configmaps

services

endpoints

crontabs

deployments

nodes

### Verbs list

get

list

watch

create

update

patch

delete

## Bearer Token

Now we need to find token we can use to log in. Execute following command:

    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

It should print something like:

    Name:         admin-user-token-6gl6l
    Namespace:    kube-system
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name=admin-user
                kubernetes.io/service-account.uid=b16afba9-dfec-11e7-bbb9-901b0e532516

    Type:  kubernetes.io/service-account-token

    Data
    ====
    ca.crt:     1025 bytes
    namespace:  11 bytes
    token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTZnbDZsIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiMTZhZmJhOS1kZmVjLTExZTctYmJiOS05MDFiMGU1MzI1MTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.M70CU3lbu3PP4OjhFms8PVL5pQKj-jj4RNSLA4YmQfTXpPUuxqXjiTf094_Rzr0fgN_IVX6gC4fiNUL5ynx9KU-lkPfk0HnX8scxfJNzypL039mpGt0bbe1IXKSIRaq_9VW59Xz-yBUhycYcKPO9RM2Qa1Ax29nqNVko4vLn1_1wPqJ6XSq3GYI8anTzV8Fku4jasUwjrws6Cn6_sPEGmL54sq5R4Z5afUtv-mItTmqZZdxnkRqcJLlg2Y8WbCPogErbsaCDJoABQ7ppaqHetwfM_0yMun6ABOQbIwwl8pspJhpplKwyo700OSpvTT9zlBsu-b35lzXGBRHzv5g_RA

Now copy the token and paste it into Enter token field on log in screen. 

Click Sign in button and that's it. You are now logged in as an admin.