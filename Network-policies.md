# Network Policies

## INTRODUCTION
Network Policies is a new Kubernetes feature to configure how groups of pods are allowed to communicate with each other and other network endpoints. In other words, it creates firewalls between pods running on a Kubernetes cluster.

By default, Kubernetes does not restrict traffic between pods running inside the cluster. This means any pod can connect to any other pod as there are no firewalls controlling the intra-cluster traffic.

Network Policies give you a way to declaratively configure which pods are allowed to connect to each other. These policies can be detailed: you can specify which namespaces are allowed to communicate, or more specifically you can choose which port numbers to enforce each policy on.

## CURRENT SETUP
In the current setup, network policies are being set when a namespace is created.

Repo: https://code.pruconnect.net/projects/RTSREAR/repos/aks-namespace/browse

By default, a policy is being set to deny all ingress traffic from other namespaces, and some exclusions are included in this deployment, which are detected via labels.

### Default Exclusions:
- Prometheus
- Node Exporter
- Kube State Metrics
- Grafana
- NGINX
- Alert Manager

To add more default exclusions, one needs to edit the `main.tf` file in the `var` folder with the following format:

#### Example format
```yaml
- name: alertmanager-mesh
  ingress:
    expressions: #insert labels here 
    - key: app #app: alertmanager
      operator: In
      values:
      - alertmanager
    - key: alertmanager # alertmanager: main
      operator: In
      values:
      - main
  ports:
  - number: 9094
    protocol: TCP
  - number: 9094
    protocol: UDP
  labels:
    alertmanager: main
