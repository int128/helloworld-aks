# helloworld-aks

Hello World with Azure AKS.

```console
% brew install azure-cli

% az login

% az aks get-credentials -n helloworld -f kubeconfig
% export KUBECONFIG=kubeconfig

% kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-agentpool-11798053-vmss000000   Ready    agent   9m3s   v1.15.10

% kubectl get pods -A
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-698c77c5d7-42vx6                1/1     Running   0          12m
kube-system   coredns-698c77c5d7-7npqk                1/1     Running   0          8m48s
kube-system   coredns-autoscaler-5bd7c6759b-qbzk8     1/1     Running   0          12m
kube-system   kube-proxy-fl2dc                        1/1     Running   0          9m9s
kube-system   kubernetes-dashboard-74d8c675bc-8w6s4   1/1     Running   1          12m
kube-system   metrics-server-7d654ddc8b-6ztpj         1/1     Running   1          12m
kube-system   omsagent-rs-6d788f7bfd-n8l7r            1/1     Running   0          12m
kube-system   omsagent-rv7xg                          1/1     Running   0          9m9s
kube-system   tunnelfront-79f9f9bd89-9zxmb            1/1     Running   0          12m
```

