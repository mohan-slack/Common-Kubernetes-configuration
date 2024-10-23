# Common-Kubernetes-configuration
Common Kubernetes configuration README

When setting up a new Kubernetes cluster (AKS or GKE), there are several configurations that are applied to ensure proper functionality and security. This document provides an overview of these configurations, so you understand what is available in your cluster.

  Register Cluster in Deployment Service
  Pre-configure RBAC
  Configure Priority Classes
  Configure Storage Classes
  Configure common secrets syncer
  Install Vault Agent Injector
  Install Ingress Controller
  Configure Fabric layer

##**Register Cluster in Deployment Service**
The first step in k8s cluster configuration is to register the cluster in the Deployment service (ArgoCD).

This service automates configuration management using Helm charts, which are a convenient packaging format for k8s applications.

In addition to this, ArgoCD provides observability for live applications running in k8s clusters.

##**Pre-configure RBAC**
Helm chart	rt-sre-rbac
Role-Based Access Control (RBAC) is a key security feature of Kubernetes that regulates access to the cluster's resources based on defined roles.

This chart adds RBAC configurations for two groups of users:

company-cluster-admins
company-cluster-viewers

##**Configure Priority Classes**
Helm chart	rt-sre-priority-classes
Prior to deploying any workloads, we configure Priority Classes for the cluster.

Helm chart creates a priority class named cluster-configuration-high which ensures that all cluster configuration service pods marked with this priority class are running.

##**Configure Storage Classes**
Helm chart	rt-sre-storage-classes
Storage Classes are used to dynamically provision storage volumes for k8s workloads.

The following Storage Classes are created for GKE:

company-disk
company-disk-encrypted
company-ssd-disk
company-ssd-disk-encrypted
company-disk-encrypted-wffc
company-ssd-disk-encrypted-wffc
And for AKS:

prudential-disk
prudential-files

##**Configure common secrets syncer**
Helm chart	rt-sre-kubed
To use Company's internal services such as the Image Registry and Secret Management, we pre-configure the Common Secrets Syncer.

This syncer keeps secrets in sync between k8s namespaces, so all workloads can expect them in namespace.

##**Install Vault Agent Injector**
Helm chart	rt-sre-vault-agent-injector
The Vault Agent Injector is a mutation webhook that injects secrets from Hashicorp Vault into your cluster's pods.

This service reads K/V secrets from Vault and injects them into the running pods, ensuring secure access to sensitive information.

This allows us to manage secrets in a secure and centralized manner.

##**Install Ingress Controller**
Helm chart	rt-sre-ingress-nginx
The Ingress Controller is used to handle incoming traffic to the k8s cluster.

It is configured for development activities and is also capable of obtaining an SSL certificate for the internal domain.

The internal domain for development activities registered in Prudential DNS and is uniq for each cluster.

# e.g. *.lb1-lbuname-dev-az1-y1am8l.ibmu.intranet.asia
*.lb1-<lbu>-<stage>-<location>-<appref>.ibmu.intranet.asia

##**Configure Fabric layer**
The Fabric layer is a place for custom resource definitions (CRDs) and operators to be deployed. For example, you may use the Couchbase Operator or Druid CRDs in your cluster.

These operators and CRDs help you manage your applications more easily and effectively.

The Fabric Layer provides a centralized place for managing these resources and helps ensure consistency.
