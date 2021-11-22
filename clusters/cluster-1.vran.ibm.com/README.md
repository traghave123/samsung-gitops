# GEMINI

Gemini is a RAN workload cluster.  Gemini is meant to be deployed by a Management cluster.

## Networks
Domain: gemini.altiostar.lan

| IP or CIDR     | USAGE        |
|----------------|:-------------|
| 172.16.0.0/24  | Cluster CIDR |
| 172.16.0.98    | apiVIP       |
| 172.16.0.99    | ingressVIP   |
| 172.16.0.100   | Reserved     |

## NODES

| SERVER        | FQDN                          | LABEL     | NODE IP      | SERVICE TAG | BMC IP        | LOCATION  |
|---------------|-------------------------------|-----------|--------------|-------------|---------------|-----------|
| DELL Server 4 | gemini01.gemini.altiostar.lan | GEMINI-02 | 172.17.0.151 | JM4L673     | 10.21.248.202 | LDC1      |
| DELL Server 5 | gemini02.gemini.altiostar.lan | GEMINI-03 | 172.17.0.152 | JM4Q673     | 10.21.248.206 | LDC1      |
| SMCI Server 1 | gemini11.gemini.altiostar.lan | GEMINI-11 | 172.18.0.151 |             | 10.27.1.77    | FEC1      |
| SMCI Server 2 | gemini12.gemini.altiostar.lan | GEMINI-12 | 172.19.0.152 |             | 10.27.1.74    | FEC2      |
| SMCI Server 5 | gemini13.gemini.altiostar.lan | GEMINI-13 | 172.18.0.153 | NA          | NA            | FEC5      |

## CONTROL PLANE Virtual Machines (VMs) -- Control Plane on Lab

| gemini-master(s)                    | MAC               | IP           | VM CONFIG                  |
|-------------------------------------|-------------------|--------------|----------------------------|
| gemini-master1.gemini.altiostar.lan | 52:52:00:11:33:11 | 172.16.0.101 | 8 vCPU, 16G RAM, 120G Disk |
| gemini-master2.gemini.altiostar.lan | 52:52:00:11:33:12 | 172.16.0.102 | 8 vCPU, 16G RAM, 120G Disk |
| gemini-master3.gemini.altiostar.lan | 52:52:00:11:33:13 | 172.16.0.103 | 8 vCPU, 16G RAM, 120G Disk |

# Cluster Deployment using Hive / Metal3 / OpenShift Infrastructure Operator

```bash
 cd ztp/
 ```

```bash
 oc kustomize .
 ```
 to examine what would be applied.

```bash
oc apply -k .
```

to apply manifests using Kustomize in this directory.

```bash
oc create -f 02-bmh-ran-gemini-altiostar-lan-RWN.yaml
```

## Verification During RAN Cluster Buildout
After feeding the ztp/02-bmh* CRD file to the management cluster, the workload cluster will start deploying.  To check:

``` bash
$ oc get agents.agent-install.openshift.io -o wide
TBD
```

### Check node booting and associating with OpenShift Infrastructure operator
This command can be used to get regular status from the cluster deployment:

```bash
oc get agents.agent-install.openshift.io -n assisted-installer  -o=jsonpath='{range .items[*]}{"\n"}{.spec.clusterDeploymentName.name}{"\n"}{.status.inventory.hostname}{"\n"}{range .status.conditions[*]}{.type}{"\t"}{.message}{"\n"}{end}'

TBD
```

### Commands to get kubeconfig and kubeadmin password
Use the following two commands once the cluster is finished buiding to retrieve both the kubeconfig and kubeadmin credentials:

```bash
oc get secret -n assisted-installer cluster-ran-gemini-altiostar-lan-admin-kubeconfig -o json | jq -r '.data.kubeconfig' | base64 -d > ~/kubeconfig-gemini
oc get secret -n assisted-installer cluster-ran-gemini-altiostar-lan-admin-password -o json | jq -r '.data.password' | base64 -d > ~/kubeadmin-gemini
```

## Post deployment
The following manifests configure the cluster to operate as a RAN cluster.  You will see some warning messages and some "unable to recognize" and Error from server (Invalid) messages.  Run `oc apply -k` a few times.

```bash
export KUBECONFIG=~/kubeconfig-skylark
export TELCO_RANCLUSTER_PATH=~/rhocp-castor/clusters/gemini.altiostar.lan
oc kustomize $TELCO_RANCLUSTER_PATH

oc apply -k $TELCO_RANCLUSTER_PATH
```
