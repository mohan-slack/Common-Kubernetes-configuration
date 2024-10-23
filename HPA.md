Created by Melvin Eng Heok Lau, last modified on Oct 24, 2019




Cluster Autoscaling
This is enabled in the AKS Terraform Provisioning script.

CA watches for pods that can't be scheduled on nodes because of resource constraints. The cluster then automatically increases the number of nodes.

Can be affected by:
The max pods per node defined during the cluster creation
Node resource utilisation is high (e.g. high CPU or RAM utilisation)


Horizontal Pod Autoscaler
This needs to be deployed together with an app.

HPA uses the Metrics Server in a Kubernetes cluster to monitor the resource demand of pods. If an application needs more resources, the number of pods is automatically increased to meet the demand. It can also make use of custom metrics that are reported to custom metrics servers (e.g. Prometheus, MS Azure Monitor) through the use of a custom metrics adapter and scale based on these custom metrics. Custom metrics can vary based on different applications, so it is up to the team to evaluate and decide which custom metric should be used for a particular application (e.g. queue length or average request response time, instead of http requests per second).

This guide will cover resource metrics and custom metrics. One thing to note is that there is another type of metrics called external metrics (Metrics that are coming from outside the K8S cluster).

Steps to enable HPA (Resource Metrics)
Is the metric-server service installed in your K8S cluster?
kubectl get svc metric-server -n kube-system
This is automatically installed in AKS Clusters 1.10 or later.
If using a lower version
git clone https://github.com/kubernetes-incubator/metrics-server.git
kubectl create -f metrics-server/deploy/1.8+/
Declarative method
Include the following yaml in your deployment
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: <name of app>
  namespace: <insert namespace>
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: <name of deployment>
  minReplicas: 
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
kubectl method
kubectl autoscale deployment <name of deployment> --cpu-percent=50 --min=3 --max=10
Check status of hpa
kubectl get hpa -n <namespace>
You should see the entry that you created above in the resulting list.


Steps to enable HPA (Custom Metrics) - Prometheus Monitoring

This section assumes that the custom metrics will be scrapped by Prometheus, and the target custom metric that will affect the hpa is http_requests_per_second, which is a derivative of a metric that is reported from the app.

Is there an instance of Prometheus already available?
If not available
Create a directory prometheus : $mkdir prometheus
Create the following files with the touch <filename> command in the directory:
prometheus.yaml
prometheus-cluster-role-binding.yaml
prometheus-cluster-role.yaml
prometheus-service-account.yaml
File contents
  prometheus.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  labels:
    prometheus: prometheus
spec:
  replicas: 1
  scrapeInterval: 30s
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      app: sample-app #change this to the label(s) that your app uses

 prometheus-cluster-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: default

 prometheus-cluster-role.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]

 prometheus-service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus

Now move out of the directory and use the following command to install prometheus in your cluster
kubectl apply -f prometheus --namespace <target namespace>
Setup Prometheus Adapter
The Prometheus Adapter is a K8S adapter that will gather the names of the available metrics from Prometheus and expose these metrics through the K8S custom metrics API.
Add the following to aks-cluster-configuration/files/k8s-prometheus/custom-metrics-config-map.yaml, under the rules: block
  Contents
rules:

  default: false
  custom:
  - seriesQuery: 'http_requests_total{namespace!="",pod!=""}'
    resources:
      overrides:
        namespace: {resource: "namespace"}
        pod: {resource: "pod"}
    name:
      matches: "^(.*)_total"
      as: "${1}_per_second"
    metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
Replace the following
url: URL to your Prometheus server
seriesQuery: A Prometheus series statement that can be seen from your Prometheus server dashboard
metricsQuery: A Go template that gets formed into a Prometheus query
Reapply the Prometheus adapter with the above yaml manifest
(Optional) To verify that the prometheus adapter can access the custom metrics from Prometheus
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
$ kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1 | jq .

{

  "kind": "APIResourceList",

  "apiVersion": "v1",

  "groupVersion": "custom.metrics.k8s.io/v1beta1",

  "resources": [

    {

      "name": "pods/http_requests_per_second",

      "singularName": "",

      "namespaced": true,

      "kind": "MetricValueList",

      "verbs": [

        "get"

      ]

    },

    {

      "name": "namespaces/http_requests_per_second",

      "singularName": "",

      "namespaced": false,

      "kind": "MetricValueList",

      "verbs": [

        "get"

      ]

    }

  ]

}

Deploy your app and Service Monitor to allow Prometheus to detect and scrape your app's metrics
 Example (sample-app.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  labels:
    app: sample-app
    team: rps-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
      team: rps-demo
  template:
    metadata:
      labels:
        app: sample-app
        team: rps-demo
    spec:
      containers:
      - image: luxas/autoscale-demo:v0.1.2
        name: metrics-provider
        ports:
        - name: http
          containerPort: 8080

  Example (sample-app-service-monitor.yaml)
kind: ServiceMonitor
apiVersion: monitoring.coreos.com/v1
metadata:
  name: sample-app
  labels:
    app: sample-app
spec:
  selector:
    matchLabels:
      app: sample-app
      team: rps-demo
  endpoints: 
  - port: http

Ensure that the labels in matchLabels match those of your app

Ensure that the port field matches the port name of your app

Apply your HPA
  Example (sample-app-hpa.yaml)
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2beta1
metadata:
  name: sample-app
spec:
  scaleTargetRef:
    # point the HPA at the sample application
    # you created above
    apiVersion: apps/v1
    kind: Deployment
    name: sample-app
  # autoscale between 1 and 10 replicas
  minReplicas: 1
  maxReplicas: 500
  metrics:
  # use a "Pods" metric, which takes the average of the
  # given metric across all pods controlled by the autoscaling target
  - type: Pods
    pods:
      # use the metric that you used in the Custom Metric API
      metricName: http_requests_per_second
      # target 500 milli-requests per second,
      # which is 1 request every two seconds

      # Current setting 1000 requests per second
      targetAverageValue: 1000000m
