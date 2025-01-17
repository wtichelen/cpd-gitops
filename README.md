# IBM Cloud Paks - GitOps Demo

## Contents

- [IBM Cloud Paks - GitOps Demo](#ibm-cloud-paks---gitops-demo)
  - [Contents](#contents)
    - [IBM Cloud Paks](#ibm-cloud-paks)
    - [Shared cluster](#shared-cluster)
    - [GitOps](#gitops)
    - [Governance Policies](#governance-policies)
  - [Hardware](#hardware)
    - [Hardware support constraints](#hardware-support-constraints)
  - [Storage](#storage)
  - [Installation](#installation)
    - [Individual clusters](#individual-clusters)
    - [Fleet of clusters with governance](#fleet-of-clusters-with-governance)
  - [Contributing](#contributing)

This repository contains Argo CD `Application` resources representing sample deployments of IBM Cloud Paks, and, as such, they are meant for inclusion in an Argo CD cluster. Different Cloud Paks are represented with different `Application` resources and grouped by a resource label tied to each Cloud Pak.

**Important**: This repository demonstrates how Cloud Pak deployments can be deployed and managed with GitOps practices. Adoption in a production environment can start from a repository fork, followed by customization of folders and files to match the desired configuration.

You may decide to include one or more of these `Application` objects to the target cluster and then determine which ones you want to synchronize into the cluster.

### IBM Cloud Paks

[IBM Cloud® Paks](https://www.ibm.com/cloud/paks) help organizations build, modernize, and manage applications securely across any cloud.

The supported deployment mechanisms for Cloud Paks are documented in their respective [documentation pages](https://www.ibm.com/docs/en/cloud-paks) and typically included a UI-based deployment through the Operator Hub page or, in some cases, scripted alternatives based on command-line interfaces.

Supported versions:

| Cloud Pak | Version | Installation mode |
| ----------|---------|-------------------|
| Cloud Pak for Business Automation | [23.0.2](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2) | Multi-pattern starter deployment |
| Cloud Pak for Data | [4.8.4](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=overview) | Online, specialized installation |
| Cloud Pak for Integration | [2023.4](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2023.4) | Online installation |
| Cloud Pak for Security | [1.10.15](https://www.ibm.com/docs/en/cloud-paks/cp-security/1.10) | Online installation |
| Cloud Pak for AIOps\* | [4.3.0](https://www.ibm.com/docs/en/cloud-paks/cloud-pak-watson-aiops/4.3.0) | Starter Installation |

\* Starting with release 4.2.0, Cloud Pak for Watson AIOps was renamed Cloud Pak for AIOps, which required renaming several folders and labels from `cp4waiops` to `cp4aiops`. You can see all changes in [pull request #281](https://github.com/IBM/cloudpak-gitops/pull/281).

### Shared cluster

Starting with the v0.22 release, deploying Cloud Paks using dedicated automation foundation instances is possible.

At the root of this configuration lies a pre-synchronization hook inside the `cp-shared` application, which creates a default "common-service-maps" ConfigMap under the `kube-public` namespace, according to the instructions listed under <https://www.ibm.com/docs/en/cloud-paks/1.0?topic=cfs-installing-cloud-pak-foundational-services-in-multiple-namespaces>

Note that if you want to enable this feature and customize the target namespaces for Cloud Paks, you must update the parameters to the `cp-shared-app` application to override that default location.

### GitOps

GitOps is a declarative way to implement continuous deployment for cloud-native applications. The Red Hat® OpenShift® Container Platform offers the [OpenShift GitOps operator](https://docs.openshift.com/container-platform/4.13/cicd/gitops/understanding-openshift-gitops.html), which manages the entire lifecycle for [Argo CD](https://argoproj.github.io/argo-cd/) and its components.

### Governance Policies

Practicing GitOps at scale, with dozens or even hundreds of clusters, benefits from a level of abstraction where each cluster follows a few select policies. This repository contains a simple deployment of governance policies for deploying OpenShift GitOps and Cloud Paks to a fleet of clusters.

## Hardware

Within the hardware support constraints (see below) of each Cloud Pak, the Argo CD applications in this repository were tested with OpenShift clusters running in `amd64` and `ppc64le` processor architecture.

The repository content is enabled to run in the `s390x` architecture but has not been directly tested with it.

### Hardware support constraints

- The API Connect component of Cloud Pak for Integration is not supported in the `ppc64le` processor architecture.
- The R Studio component of Cloud Pak for Data is only supported in `amd64` processors.
- The AI Manager component of Cloud Pak for AIOps is only supported in `amd64` processors.
- Cloud Pak for Security is only supported in `amd64` processors.

This section's list of exclusions and support is not exhaustive. Always consult the respective Cloud Pak documentation to determine if a component is supported in a given hardware platform.

## Storage

The instructions in this repository assume the user already has an OpenShift cluster with storage capable of RWO and RWX access mode.
The Argo CD `Application` resources have pre-synchronization hooks that will attempt auto-detection of the storage in the cluster from common storage providers in decreasing order of precedence:

| Precedence | OpenShift platform | Storage | Storage classes |
| ---------- | ------------ | ------- | --------------- |
| Highest    | All          | OpenShift Data Foundation | `ocs-storagecluster-ceph-rbd` (RWO) and `ocs-storagecluster-cephfs` (RWX). |
| High       | All          | Rook Ceph | `rook-ceph-block` (RWO) and `rook-cephfs` (RWX) |
| Medium     | All          | NetApp ONTAP (Trident driver) | `trident-csi` (RWO and RWX.) Note about RWO access mode: the synchronization hooks prefer a storage class named `trident-block-csi` if they can find one in the cluster. |
| Low        | IBM Cloud (ROKS classic) | IBM Cloud File Storage | `ibmc-block-gold` (RWO) and `ibmc-file-gold-gid` (RWX) |
| Low        | AWS (self-managed and ROSA) | Elastic Block Store and Elastic File System | First storage class with provisioner `ebs.csi.aws.com` (RWO) and first storage class with provisioner `efs.csi.aws.com` (RWX) |
| Low        | Azure (self-managed and ARO) | Azure Disk (classic) and Azure File System | `managed-premium` (RWO) and `azure-file` (RWX) |

Note that this is a list of validated storage providers for this repository.
Cloud Paks may support different matrixes of storage vendors.

## Installation

### Individual clusters

Argo applications are added to the Argo CD server. An application defines the source of the Kubernetes resources and the target cluster where those resources should be deployed. The Argo CD server "installs" a Cloud Pak by synchronizing the applications representing the Cloud Pak into the target cluster.

Refer to the [installation page](docs/install.md) for instructions on configuring an OCP server with the OpenShift GitOps operator and then adding the Cloud Pak GitOps resources to the default GitOps server created by the operator.

### Fleet of clusters with governance

Use governance policies and placement rules to configure entire clusters with GitOps infrastructure and manage Cloud Pak deployments from Red Hat Advanced Cluster Management for Kubernetes (RHACM.)

Refer to the [RHACM page](docs/rhacm.md) for an overview and instructions on how to add RHACM to an existing OpenShift cluster.

## Contributing

Making changes to this repository requires a working knowledge of Argo CD administration and configuration. This section describes the workflow of submitting a change. A change entails forking the repository, modifying it, installing the changes on a target cluster to validate them, then gathering the output of validation commands using the `argocd` command-line interface.

Navigate to the [contributing](CONTRIBUTING.md) page for details on validating changes and submitting a pull request with all the required information.
