# Directory: clusters

This directory contains artifacts best used with the kubernetes tool `kustomize` or tooling that utilizes a GitOps methodology like Red Hat Advanced Cluster Manager for Kubernetes (RHACM).

## Terminology

- Isolated Network - one that requires the usage of a local container registry and one that cannot get out to Red Hat repositories (registry.redhat.io, quay.io, etc) directly
- Restricted network - one that requires the usage of a proxy to access external redhat container repositories.

## Extending directory structure

Todo: Update this section.

| Directory                                     | Description/Usage                                                                                 |
|-----------------------------------------------|---------------------------------------------------------------------------------------------------|
| `/clusters/base`                              | contains unique elements applicable to all OpenShift clusters.                                    |
| `/clusters/base/ran`                          | contains unique elements applicable to all OpenShift RAN clusters.                                |
| `/clusters/base/ran-cu`                       | contains unique elements applicable to all OpenShift RAN clusters containing Centralized Units.   |
| `/clusters/base/ran-du`                       | contains unique elements applicable to all OpenShift RAN clusters containing Distributed Units.   |
| `/clusters/base/rwn`                          | contains unique elements applicable to all OpenShift RAN clusters utilizing remote worker nodes.  |
| `/clusters/base/operators`                    | contains repeatable OpenShift Operator content that is applicable across OpenShift clusters.      |
| `/clusters/<cluster-name>/configs`            | Cluster specifics configurations of platform elements                                             |
| `/clusters/<cluster-name>/sites/<site-id>/`   | Configurations affecting nodes of a cluster on a specific site                                    |
| `/cnf`                                        | Configurations related to the CNF workloads                                                       |
| `/unsupported`                                | Commercially unsupported reference implementations, utilities and reference to 3rd-party projects |

## OCP Versions for Altiostar Drops

| DROP   | OCP VERSION | DATE           | INTENDED USE              |
|--------|-------------|----------------|---------------------------|
| DROP 0 | OCP 4.6.14  | January 2021   | Lab only                  |
| DROP 2 | OCP 4.6.16  | February 2021  | Lab only                  |
| DROP 3 | OCP 4.6.18  | March 30 2021  | Lab & QA                  |
| DROP 4 | OCP 4.8.x   | (TBD)          | Lab, QA & Customer Labs   |
| DROP 5 | TBD         | (TBD)          | Lab, QA & Customer Labs   |

### Upgrading the cluster

Clusters with access to the online update service (api.openshift.com) or on-premise OpenShift Update Service use the following procedure to upgrade a cluster.

```bash
# Upgrade cluter to OCP 4.8.12 (DROP 4)
oc adm upgrade --to=4.8.12

# Upgrade cluster to latest stable (future latest)
# WARNING: To consider a different stable version in a Drop
# validate release compatibility with the required Operators
oc adm upgrade --to-latest
```

NOTE: For instructions on running an on premise [OpenShift Update Servce](https://www.openshift.com/blog/openshift-update-service-update-manager-for-your-cluster) see linked blog for details.

## Cluster Directory structure

The cluster directory structure is described in [CLUSTER.md](CLUSTER.md)
