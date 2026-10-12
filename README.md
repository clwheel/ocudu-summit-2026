# OCUDU Summit 2026 - O-RAN O2-IMS Hub Cluster Configuration

This repository contains Kubernetes manifests and templates for deploying and managing an OpenShift hub cluster with O-RAN O2-IMS provisioning capabilities. The hub cluster serves as a centralized management platform for deploying edge clusters using the O-RAN O2 Interface Management Services (O2-IMS) provisioning interface.

## Overview

The configuration enables:
- **Hub Cluster Setup**: Automated deployment of OpenShift operators including ACM, GitOps, and O-Cloud Manager
- **Infrastructure as Code**: Declarative cluster templates and policies for reproducible edge cluster deployments
- **O-RAN Compliance**: O2-IMS based provisioning workflow for RAN Distributed Unit (DU) clusters
- **Hardware Abstraction**: Hardware profiles and inventory management for bare-metal deployments

## Repository Structure

### `/manifests`

Hub cluster operator and service configuration manifests, applied in numerical order:

- `00-namespaces.yaml` - Creates required namespaces (ACM, GitOps, storage, O2-IMS)
- `01-operatorgroups.yaml` - Operator group definitions
- `02-operators.yaml` - Operator subscriptions including:
  - Advanced Cluster Management (ACM) 2.17
  - Topology Aware Lifecycle Manager (TALM)
  - OpenShift Data Foundation (ODF) 4.22
  - O-Cloud Manager (O2-IMS implementation)
  - OpenShift GitOps
- `03-localvolumes.yaml` - Local storage configuration
- `04-storagecluster.yaml` - ODF storage cluster setup
- `05-multiclusterhub.yaml` - ACM MultiClusterHub configuration
- `06-mco.yaml` - ACM MultiClusterObservability configuration
- `07-provisioning.yaml` - Metal3 provisioning service
- `08-agentserviceconfig.yaml` - Assisted Installer agent service

**Purpose**: Configure the hub cluster with all necessary operators and services for managing edge cluster deployments.

### `/clustertemplates`

Reusable templates for cluster deployments via O2-IMS provisioning:

#### `hardwareprofiles/`
Hardware-specific BIOS and BMC configurations:
- `hpe-dl380-bios-v3.70-bmc-v3.21.yaml` - HPE DL380 server profile
- `kustomization.yaml` - Kustomize configuration for hardware profiles

**Purpose**: Define hardware-specific settings for consistent bare-metal deployments.

#### `inventory/`
Infrastructure inventory definitions:
- `common.yaml` - Shared infrastructure resources
- `location.yaml` - Site location metadata
- `ocloudsite.yaml` - O-Cloud site definition
- `resourcepool.yaml` - Compute resource pools
- `master-0-sno1.yaml`, `master-0-sno2.yaml` - Individual BareMetalHost inventory
- `kustomization.yaml` - Kustomize configuration for inventory

**Purpose**: Maintain infrastructure inventory for resource allocation during provisioning.

#### `ocp-4.22/`
OpenShift 4.22 cluster templates:
- `sno-ran-du/` - Single Node OpenShift (SNO) RAN DU template
  - `sno-ran-du-v4-22-14-1.yaml` - ClusterTemplate CR
  - `clusterinstance-defaults-v1.yaml` - Default cluster parameters
  - `policytemplates-defaults-v1.yaml` - Default policy bindings
  - `pull-secret.yaml` - Image pull secret
  - `ns.yaml` - Namespace definition
- `extra-manifest/` - Additional cluster manifests
- `kustomization.yaml` - Kustomize configuration

**Purpose**: Define versioned cluster templates for different OpenShift releases and deployment patterns.

### `/policytemplates`

ACM PolicyGenTemplate definitions for post-deployment cluster configuration:

#### `source-crs/`
Source custom resources for policy generation.

#### `custom-crs/`
Custom CR definitions organized by function:
- `machineconfig/` - MachineConfig resources for node-level configuration
- `ocudu/` - OCUDU-specific configurations
- `open5gs/` - Open5GS CNF configurations
- `ptp/` - Precision Time Protocol (PTP) configurations

#### `sno-ocudu/`
Complete policy template for SNO OCUDU deployments combining:
- SR-IOV networking
- Performance tuning (CPU isolation, hugepages)
- PTP configuration
- CNF-specific settings

**Purpose**: Provide declarative, version-controlled policies for Day 2 cluster configuration applied via ACM.

### `/provisioningrequests`

Example O2-IMS ProvisioningRequest CRs:
- `sno1.yaml`, `sno1.json` - Provisioning request for first SNO cluster
- `sno2.yaml`, `sno2.json` - Provisioning request for second SNO cluster

Each request includes:
- Cluster template reference (e.g., `sno-ran-du`)
- Template version (e.g., `v4-22-14-1`)
- Hardware profile assignment
- Node networking configuration
- Policy template parameters (SR-IOV, CPU tuning, hugepages)
- Cluster instance parameters (DNS, NTP, domain)

**Purpose**: Demonstrate complete cluster provisioning workflow through the O2-IMS API.

## Workflow

1. **Hub Setup**: Apply manifests in `/manifests` to deploy the hub cluster infrastructure
2. **Template Registration**: Deploy cluster templates from `/clustertemplates` to the hub
3. **Policy Registration**: Deploy policy templates from `/policytemplates` to ACM
4. **Cluster Provisioning**: Submit ProvisioningRequests to the O2-IMS API referencing templates
5. **Automated Deployment**: O-Cloud Manager orchestrates cluster deployment and policy application

## Prerequisites

- OpenShift 4.22+ hub cluster
- Bare-metal infrastructure with BMC access
- Access to Red Hat operator catalogs
- Valid OpenShift pull secret

## Key Technologies

- **OpenShift**: Container platform
- **Advanced Cluster Management (ACM)**: Multi-cluster management
- **O-Cloud Manager**: O-RAN O2-IMS implementation
- **GitOps**: Declarative cluster lifecycle management
- **Kustomize**: Configuration management
- **Metal3**: Bare-metal provisioning
- **Topology Aware Lifecycle Manager (TALM)**: Policy orchestration

## References

- [O-RAN O2 Interface Specification](https://www.o-ran.org/specifications)
- [OpenShift Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform)
- [ACM Documentation](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/)
- [O-Cloud Manager Documentation](https://github.com/openshift-kni/oran-o2ims)
