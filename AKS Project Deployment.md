New Project
Create Namespace
Secret Sources
AppRole Access
Privileged Access
Namespace Security Policy
Application Gateway Configuration
Azure Firewall Whitelisting
Update Project
Project Decommission
Application Deployment
Create Application
Update Application
Delete Application
Helm charts deployment
Secrets in Hashicorp Vault
Project Owner Access
Project Deployer Token and AppRole
New Project
Navigate to https://code.pruconnect.net/projects/RTSREAN/repos/aks-namespace-request/browse
Fork the repo into a personal project
Edit namespace.yml in the template branch as required. Follow the guide below
Raise a pull request against the master branch
Once the pull request is merge, a deployment job will get automatically triggered
After deployment, an email will be sent to the project owner with the details 
Create Namespace
This is the minimal requirement for the project deployment.

Note that this is in YAML syntax. The deployment will fail if the yaml structure is not followed.

---
# This is the 6-character application reference of the cluster.
# The LBU, environment and subscription are all derived from this.
# Select the app_ref from https://core.pru.intranet.asia/kubernetes_engine/clusters/
app_ref: abc123
 
# The location of the cluster.
#
# az1 for Southeast Asia
# az2 for East Asia
#
# Note that a cluster may not be available on both locations
 
location: az1
 
 
# The name of the namespace.
# Restrict this to lower case alphanumeric characters, -, and .
# Note that final name will be prefixed by the '<lbu>-<environment>-<location>-'
# e.g. sgrtss-dev-az1-namespace-demo
 
name: namespace-demo
 
 
# The environment of the namespace
# dev/uat/sit for non-production
# prd for production
 
environment: dev
 
 
# The email address of the namespace owner. The owner also becomes a namespace owner.
# The deployer token will be stored in HSV.
# Enter the whole email address
 
owner: namespace.owner@prudential.com.sg
 
 
# A list of email addresses of the persons who can read the namespace
# Enter the whole email address for each entry
 
admins:
  - admin@prudential.com.sg
  - first.last@prudential.com.sg
 
 
# A list of email addresses of the persons who can only view the namespace
# Enter the whole email address for each entry
 
viewers:
  - viewer@prudential.com.sg
  - first.last@prudential.com.sg
Secret Sources
By default, the namespace service account will have access to kv2/<lbu>/<environment>/<subscription>/<app_ref>/<az1|az2>/<namespace_name> in Hashicorp Vault.

There is an option to specify additional app_ref endpoints where the application needs access to.

 Collapse source
# This will give the namespace service access to kv2/sgrtss/nprd/dev/t3stan/redis-rds-sgrtss-dev-az1-t3stan.redis.cache.windows.net
# Specify the keys that you need in the application deployment step below
 
secret_sources:
  - app_ref: t3stan
    paths:
      - redis-rds-sgrtss-dev-az1-t3stan.redis.cache.windows.net
AppRole Access
The Hashicorp Vault approle is similar to a service account that allows one to access the secret endpoint for the namespace.

However, logging in to Vault using an approle can only be done from a host within a list of subnets.

 Collapse source
# This is a comma-separated list of subnets in CIDR notation
# RT Jenkins CIDRs are already added by default
 
approle_cidrs: ['123.456.789.0/32']
Privileged Access
For applications that require a privileged security context (e.g. run as root), refer to the option below

Note that this is not required and discouraged in most cases

Enabling this will require a business justification and approval from the securities team

 Collapse source
privileged: true
Namespace Security Policy
By default, all traffic coming into the namespace from another namespace are denied

To exclude some policies from being denied, add and modify the entries below

Note that this is not required and discouraged in most cases

Enabling this will require a business justification and approval from the securities team

 Collapse source
# Sample policy
# Edit accordingly
 
network_policy: - array of network policy exclusions
  - name - Name of exclusion
    - ingress:
      - expressions: - (Optional) Array of pod selector requirements
        - key - "key" of the pod label
        - operator: In - Always set to In
        - value - "value" of the pod label
      - ports: - (Optional) List of ports to monitor
        - number - Port number to allow
        - protocol - (Optional) TCP / UDP
    - labels - list of labels for the pod selector
Application Gateway Configuration
To configure application gateway to listen to an external url and use AKS endpoints as backend pools, add the following configuration and edit accordingly.

 Collapse source
inbound_urls:
  # the external url fqdn
  - fqdn: staticwebpage.prudential.com.sg
 
    # this is optional and defaults to /
    source_path: /web/
 
    # the list of ingress endpoints
    # this will be automatically suffixed by the namespace's default ingress domain (e.g. django.aks-lb1-sgrtss-uat-az1-dzwo62.pru.intranet.asia)
    ingresses:
      - django
 
    health_probe:
      # defaults to /
      # change this if the path of the backend probe is different
      path: /
 
      # indicate this if the health probe ingress is different from the 'ingresses' above
      override_ingress: django-override
     
    # this is optional and defaults to 30 seconds
    # the number of seconds that the application gateway will wait to receive a response from the backend pool
    #timeout: 60
 
    ## this is optional
    ## request_header is a string with key=value format. remove the value to delete the header
    ## response_header is a string with key=value format. remove the value to delete the header
    #rewrite_rules:
    #  - name: delete-method-override
    #    sequence: 200
    #    response_header: X-Http-Method-Override=
 
    # support for url-based routing
    routes:
      # the url path to reroute to another endpoint (e.g. staticwebpage.prudential.com.sg/images/)
      - source_path: /images/
 
        # the list of ingress endpoints
        # this will be automatically suffixed by the namespace's default ingress domain (e.g. django-web.aks-lb1-sgrtss-uat-az1-dzwo62.pru.intranet.asia)
        ingresses:
          - django-web
 
        # this is optional and defaults to 30 seconds
        # the number of seconds that the application gateway will wait to receive a response from the backend pool
        #timeout: 60
 
        ## this is optional
        ## request_header is a string with key=value format. remove the value to delete the header
        ## response_header is a string with key=value format. remove the value to delete the header
        #rewrite_rules:
        #  - name: delete-method-override
        #    sequence: 200
        #    response_header: X-Http-Method-Override=
 
        health_probe:
          # defaults to /
          # change this if the path of the backend probe is different
          path: /
 
          # indicate this if the health probe ingress is different from the 'ingresses' above
          override_ingress: django-override
Azure Firewall Whitelisting
For the AKS applications within the namespace to communicate with external urls, add the following configuration and edit accordingly.

 Expand source
Update Project
After successful deployment of the project, an email will be sent to the application owner with the details of the project.

This email contains a link to a Bitbucket repository that will become the "source of truth" for the project.

Sample:



This repository contains a project.yml with the same entries entered during the project creation.

All the project configuration should be added/updated/deleted through this file.

Once the pull request is merged to master, a deployment job will run to apply the changes accordingly.

Project Decommission
To delete the namespace and all the references to it, add the following entry to project.yml, commit the change, raise a pull request, and merge.

 Expand source
Application Deployment
Everything should be done in the project repository that was created during the project creation above.

This deployment mechanism has limited functionality and is intended for very simple use-cases.

If you require more advanced deployments, get your deployment token through Kubernetes Project - Deployer Object and implement your own deployments.

Create Application
Create a YAML file under <repo_root>/applications/ for each application to be deployed in the namespace
git add each file
git commit
Raise a pull request against the master branch
The project owner, admins and viewers should have access to review the pull request
Once the pull request is merged, a deployment job will automatically run
After deployment, an email will be sent to the project owner with the details 
Follow the guide below for the content of the YAML file.

NOTE: ALL *.yml and *.yaml files are processed! Do not leave an empty file (e.g. the template) along with valid ones as it will fail.

---
# This is the name of the deployment config and the pods created from it,
# as well as the value for the label 'app.kubernetes.io/application' which every object created from this file will get
 
name: myapp
 
 
# Replicas is the number of identical pods that will be created
# For development 1 is usually good,
# while production will usually require at least 3 for high availability.
 
replicas: 1
 
 
# The image for the containers and the tag to be deployed. The tag below is expected to be found at the source registry.
# Note that a common imagePullSecret will be used to pull this image from Artifactory (*.pruregistry.intranet.asia:8443)
 
image: docker-release.pruregistry.intranet.asia:8443/com.prudential.awb/my-app
 
image_tag: 147
 
 
 
############################################
# The following parameters are all optional.
# Comment out or remove when not needed.
############################################
 
# Use this to override the ENTRYPOINT command set in the image
# Note that both should be in list format
# https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
 
entrypoint_override:
  command:
    - sleep
  args:
    - 6000
 
 
# This will be the resources given to the container. Same value is set for requests and limits.
# https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-cpu
# https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory
 
container_resources:
  cpu: 300m
  ram: 200Mi
 
 
# Here you can add simple key:value environment variables that will be passed as a container spec
# https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/
 
env_vars:
  FOO: BAR
  very: nice
 
# Optionally, add a YAML file under 'files/' directory with the key:value mapping of the environment variables
 
env_vars_files:
  - common_vars.yml
 
 
# These secrets will be extracted from Hashicorp Vault and automatically fetched by a sidecar that runs together with the application pod
#
# static secrets will be fetched from kv2/<lbu>/<environment>/<subscription>/<app_ref>/<az1|az2>/<namespace_name>
# e.g. kv2/sgrtss/nprd/uat/dzwo62/az1/sgrtss-dev-az1-demo
# The secrets will be written into the container's filesystem under /etc/secrets/<path>/<key>
#
# Follow the "Secrets in Hashicorp Vault" section below to enter the secrets into Hashicorp Vault
 
secrets:
  static:
    - path: django
      keys:
        - DJANGO_SECRET_KEY
        - DATABASE_PASSWORD
    - path: ldap
      keys:
        - LDAP_BIND_PASSWORD
    - path: vault
      keys:
        - VAULT_TOKEN
        - VAULT_ROLE_ID
        - VAULT_SECRET_ID
 
    # Specifying the app_ref means that the secret will be read from the application specific endpoint
    # Make sure that you declared this app_ref during the namespace creation - https://collaborate.pruconnect.net/display/RTSRE/AKS+Project+Deployment#AKSProjectDeployment-SecretSources
    - app_ref: t3stan
      path: redis-rds-sgrtss-dev-az1-t3stan.redis.cache.windows.net
      keys:
        - primary_access_key
        - secondary_access_key
  
 
# These ports will be exposed as an internal cluster service
# https://kubernetes.io/docs/concepts/services-networking/service/
#
# When a route (so it can be accessed from outside the cluster)
# is required set the exposed_route field for the port as 'true'.
# When a route is exposed you can optionally force https by also
# adding secured_route: true
#
# IMPORTANT: The value for service_name cannot be more than 15 characters, and must adhere to this format:
# DNS-1123 label must consist of lower case alphanumeric characters or '-', and must start and end with an alphanumeric character
# (e.g. 'my-name', or '123-abc', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?')
 
services:
  - name: webserver
    port: 8080
    protocol: tcp
    exposed_route: true
    secured_route: true
  - name: database
    port: 9080
    protocol: udp
    exposed_route: false
 
 
# To request ephemeral directories (data will be lost when the container terminates)
# that the app can write to, simply add names and paths below
 
emptydir_volumes:
  - name: temp-data
    path: /opt/temp-data
  - name: scratch
    path: /opt/scratch
 
 
## This lets you mount keys from the configmap (i.e. the variables defined above) as files inside the container.
## You need to set a name for the volume, path to be mounted at, and the variables whose content will be in the files.
## The files will have the same name as the variables themselves.
 
configmap_volumes:
  - name: app_config
    path: /opt/foo
    keys:
      - myapp_properties
      - db_config
  - name: app_users
    path: /opt/bar
    keys:
      - myapp_users
      - dev_groups
 
 
# You can also include persistent storage backed by Azure Files or Azure Managed Disk.
# A PVC size lower than 100G will be automatically set to 100G.
# NOTE: If you REMOVE, COMMENT OUT, OR CHANGE THE SIZE of a PVC below, it will be deleted.
# Be careful if you have any data that you need to keep
 
persistent_volumes:
  - name: data
    path: /mnt/data
    size: 1Gi
    # valid values are prudential-aks-azure-files and prudential-aks-managed-disk
    # defaults to prudential-aks-azure-files
    storage_class: prudential-aks-managed-disk
  - name: database
    path: /opt/oracle
    size: 2Gi
 
# volumes:
 
 
# Declare the readiness and liveness probes
# https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
 
readiness_probe:
  - httpGet:
      path: /healthz
      port: 8080
    initialDelaySeconds: 15
    timeoutSeconds: 1
 
liveness_probe:
  - tcpSocket:
      port: 9080
    initialDelaySeconds: 15
    timeoutSeconds: 1
 
 
# Declare initContainers customization
# https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
# Be careful as customization will be injected into your Deployment configuration
# initContainers:
#   - name: example-init-container
#     image: docker-release.pruregistry.intranet.asia:8443/alpine:latest
#     command:
#       - /bin/sh
#     args:
#       - -c
#       - echo The app is starting!
#
Update Application
Edit the application details in each yaml file under <repo_root>/applications/.

Commit, raise a PR and merge the changes.

This will automatically trigger a deployment and an email will be sent to the application owner.

Delete Application
Delete the yaml file that corresponds to the application definition under <repo_root>/applications/.

Commit, raise a PR and merge the changes.

This will automatically trigger a deployment and an email will be sent to the application owner.

Helm charts deployment
You may define the charts block to provide information on what Helm charts should be installed in a namespace.

Each element of the block should have:

name
repo
revision
extra_values (optional)
charts:
  - name: echoserver
    repo: rtappchart/rt-sre-echoserver
    revision: 1.0.0
    extra_values:
      ingress.enabled: true
      ingress.baseName: lb1-sgrtss-dev-az1-spk89e.pru.intranet.asia
The repo parameter has two parts: project and repo name - so you can use your project in Bitbucket to develop a Helm chart.

Example with personal repo Expand source

If you want to create a new component, you need to read this article: Create Helm chart deployment component

Secrets in Hashicorp Vault
Project Owner Access
The project or namespace users can acquire access to secrets in Hashicorp Vault through Identity and Access Management#ManagingAccess .

Scroll to the kubernetes_namespaces section.



The project owner is automatically given access to key in the secrets into Hashicorp Vault.

Note that the project owner get access only to the endpoint specific to the namespace.

To do this, follow the Vault Use Cases Illustration (KV Secrets) article.



The secrets should be entered under kv2/<lbu>/<environment>/<subscription>/<app_ref>/<az1|az2>/<namespace_name>

Example: kv2/sgrtss/nprd/uat/dzwo62/az1/sgrtss-dev-az1-demo

Project Deployer Token and AppRole
Please refer to Kubernetes Project - Deployer Object
