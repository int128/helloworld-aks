# helloworld-aks

Hello World with Azure AKS.


## Standalone cluster

```console
% brew install azure-cli
% az login
```

```console
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


## Azure AD integration

Follow the steps in https://docs.microsoft.com/azure/aks/azure-ad-integration-cli.

If you get the following error, try regenerating the client secret.

```
% az aks create ...
Operation failed with status: 'Bad Request'. Details: Service principal client secret has invalid characters: ` '

% serverApplicationSecret=$(az ad sp credential reset \
    --name $serverApplicationId \
    --credential-description "AKSPassword" \
    --query password -o tsv)
% az ad app permission add \
    --id $serverApplicationId \
    --api 00000003-0000-0000-c000-000000000000 \
    --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope 06da0dbc-49e2-44d2-8312-53f166ab848a=Scope 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role
% az ad app permission grant --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000
% az ad app permission admin-consent --id  $serverApplicationId
```

Finally the command should success.

```console
% az aks create --resource-group myResourceGroup --name $aksname --node-count 1 --generate-ssh-keys --aad-server-app-id $serverApplicationId --aad-server-app-secret "$serverApplicationSecret" --aad-client-app-id $clientApplicationId --aad-tenant-id $tenantId
- Finished ..
```

Make sure you can access the cluster using cluster-admin.

```console
% az aks get-credentials --resource-group myResourceGroup --name $aksname --admin -f kubeconfig-admin
Merged "myakscluster-admin" as current context in kubeconfig-admin

% kubectl get nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-25325194-vmss000000   Ready    agent   12m   v1.15.10
```

kubeconfig looks like:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0t...
    server: https://myaksclust-myresourcegroup-REDUCTED.hcp.eastus.azmk8s.io:443
  name: myakscluster
contexts:
- context:
    cluster: myakscluster
    user: clusterAdmin_myResourceGroup_myakscluster
  name: myakscluster-admin
current-context: myakscluster-admin
kind: Config
preferences: {}
users:
- name: clusterAdmin_myResourceGroup_myakscluster
  user:
    client-certificate-data: LS0t...
    client-key-data: LS0t...
    token: REDUCTED
```

Make sure you can access the cluster using your own user.

```console
% az aks get-credentials --resource-group myResourceGroup --name $aksname -f kubeconfig-user
Merged "myakscluster" as current context in kubeconfig-user
```


kubeconfig looks like:

```yaml
users:
- name: clusterUser_myResourceGroup_myakscluster
  user:
    auth-provider:
      config:
        apiserver-id: REDUCTED
        client-id: REDUCTED
        config-mode: '1'
        environment: AzurePublicCloud
        tenant-id: REDUCTED
      name: azure
```

You need to authenticate on the first time.

```console
% kubectl get nodes
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code REDUCTED to authenticate.
Error from server (Forbidden): nodes is forbidden: User "REDUCTED@REDUCTED.onmicrosoft.com" cannot list resource "nodes" in API group "" at the cluster scope
```

After the command, kubeconfig looks like:

```yaml
users:
- name: clusterUser_myResourceGroup_myakscluster
  user:
    auth-provider:
      config:
        access-token: REDUCTED
        apiserver-id: REDUCTED
        client-id: REDUCTED
        config-mode: "1"
        environment: AzurePublicCloud
        expires-in: "3599"
        expires-on: "1590397783"
        refresh-token: REDUCTED
        tenant-id: REDUCTED
      name: azure
```

Add a cluster role binding to allow access from your group.

```console
% kubectl create clusterrolebinding aad-cluster-admin --clusterrole=cluster-admin --group=REDUCTED --kubeconfig=kubeconfig-admin
clusterrolebinding.rbac.authorization.k8s.io/aad-cluster-admin created
```

Make sure you can access the cluster.
