# Telco Managment Cluster

The Telco management cluster *MUST* be OpenShift 4.8.x or higher and *MUST* be deployed using one of the following methods:
- Assisted Installer / Assisted Service Operator
- Baremetal IPI
- Crucible Ansible Playbooks (coming soon)

These method are the *ONLY* methods that deploy and configure the `cluster-baremetal-operator` in the cluster which is a requirement for some of the ZTP automation flows.

## EXAMPLES

The following are examples of Telco management clusters utilizing the `telco-gitops` (github.com/openshift-telco/telco-gitops) repository:

- https://github.com/openshift-telco/telco-gitops-mgmt
- https://github.com/novacain1/carslab-public/tree/main/rhocp-clusters/mgmt/volt.cars.lab
