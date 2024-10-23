# AKS Monitoring

## AKS allows the user to monitor the cluster in various ways through the Azure Portal

### Real-Time Container Logs
Container logs can be monitored in real-time (stdout and stderr) without the need for kubectl. To do this, an RBAC is created during the cluster creation to allow reader access to pod logs and events.

The user that accesses the logs in Azure Portal is required to have Azure Kubernetes Service Cluster User Role access to the cluster resource.

### Kubernetes Master Component Logs
Kubernetes Master Component logs are being served to an Azure Logs Analytics Workspace. This is configured in the AKS Terraform Module, under the `aks-diagnostics.tf` file.

The main components that are monitored are:
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- kube-audit
- cluster-autoscaler

It can be accessed from the Logs tab in Azure Portal AKS Cluster. This allows the user to query for the logs of the main K8S components that are managed by AKS.

### Example Kusto Query
Query for cluster-autoscaler:
```kusto
AzureDiagnostics | where Category=="cluster-autoscaler" | project log_s
