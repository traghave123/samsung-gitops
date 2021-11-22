# SERIUS

Serius is a Management cluster.

# Telco Managment Cluster

The Telco management cluster must be OCP 4.8.x and be deployed using one of the following methods:
- Assisted Installer / OpenShift Infrastructure Operator mode
- Baremetal IPI mode
- Crucible Ansible Playbooks (coming soon)

These method are the only methods that deploy and configure the `cluster-baremetal-operator` in the cluster which is a requirement for some of the automation flows.

## Management Cluster Configuration

Once the cluster is operational (using one of the valid deployment methods for Telco Management Clusters), it's time to configure it.  Be sure to reference the appropriate KUBECONFIG and the path of this repository, which you should have cloned to a path appropriate to your environment.  The `oc kustomize` command will show you what will be applied.  Example is below:

```bash
export KUBECONFIG=~/kubeconfig-serius
export TELCO_MGMTCLUSTER_PATH=~/ztp-for-ocp4.8/clusters/mgmt/ocp48-controlplane.vran.ibm.com
oc kustomize $TELCO_MGMTCLUSTER_PATH
```

Below is a sample run, be sure you are referencing the appropriate kubeconfig for the Management cluster here, which in our case is ocp48-controlplane.vran.ibm.com!

```bash
$ oc whoami --show-server
https://api.ocp48-controlplane.vran.ibm.com:6443

$ cd $TELCO_MGMTCLUSTER_PATH
$ oc apply -k .
Warning: resource namespaces/assisted-installer is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by oc apply. oc apply should only be used on resources created declaratively by either oc create --save-config or oc apply. The missing annotation will be patched automatically.
namespace/assisted-installer configured
namespace/openshift-local-storage created
namespace/openshift-serverless created
namespace/telco-gitops created
serviceaccount/cli-job-sa created
clusterrole.rbac.authorization.k8s.io/cli-job-sa-role created
clusterrole.rbac.authorization.k8s.io/telco-gitops-custom-role created
clusterrolebinding.rbac.authorization.k8s.io/cli-job-sa-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/telco-gitops-cluster-admin-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/telco-gitops-custom-rolebinding created
Warning: resource schedulers/cluster is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by oc apply. oc apply should only be used on resources created declaratively by either oc create --save-config or oc apply. The missing annotation will be patched automatically.
scheduler.config.openshift.io/cluster configured
kubeletconfig.machineconfiguration.openshift.io/max-pods-500 created
machineconfig.machineconfiguration.openshift.io/50-master-create-lvs-for-lso created
machineconfig.machineconfiguration.openshift.io/50-worker-fix-ipi-rwn created
Warning: resource ingresscontrollers/default is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by oc apply. oc apply should only be used on resources created declaratively by either oc create --save-config or oc apply. The missing annotation will be patched automatically.
ingresscontroller.operator.openshift.io/default configured
operatorgroup.operators.coreos.com/assisted-service-operator created
operatorgroup.operators.coreos.com/openshift-local-storage created
operatorgroup.operators.coreos.com/openshift-serverless created
catalogsource.operators.coreos.com/assisted-service created
subscription.operators.coreos.com/assisted-service-operator created
subscription.operators.coreos.com/openshift-local-storage-operator created
subscription.operators.coreos.com/hive-operator created
subscription.operators.coreos.com/openshift-gitops-operator created
subscription.operators.coreos.com/openshift-pipelines-operator-rh created
subscription.operators.coreos.com/serverless-operator created
[unable to recognize ".": no matches for kind "Application" in version "argoproj.io/v1alpha1", unable to recognize ".": no matches for kind "ApplicationSet" in version "argoproj.io/v1alpha1", unable to recognize ".": no matches for kind "ArgoCD" in version "argoproj.io/v1alpha1", unable to recognize ".": no matches for kind "ClusterImageSet" in version "hive.openshift.io/v1", unable to recognize ".": no matches for kind "HiveConfig" in version "hive.openshift.io/v1", unable to recognize ".": no matches for kind "LocalVolume" in version "local.storage.openshift.io/v1", unable to recognize ".": no matches for kind "LocalVolumeDiscovery" in version "local.storage.openshift.io/v1alpha1"]
Error from server (Invalid): error when creating ".": StorageClass.storage.k8s.io "lso-filesystemclass" is invalid: provisioner: Required value

```
- You will see some warning messages and some "unable to recognize" and Error from server (Invalid) messages.  This is because Operators have not completed installing on the management cluster.  Run `oc apply -k` a few times.
- Disconnection for the ingressVIP and apiVIP might be experienced as the configuration is modifying the corresponding operator configuration. After a few minutes the base configuration should be completed.

- Create secret from pull-secret for the Assisted Installer Operator:

    ```bash
    oc create secret generic assisted-deployment-pull-secret -n assisted-installer \
>     --from-file=.dockerconfigjson=/home/redhat/pull-secret-original.json --type=kubernetes.io/dockerconfigjson
    ```

- Label OpenShift Supervisor nodes with the `ran.openshit.io/lso` label, so the Local Storage Operator consumes the second block device for the Openshift Infrastructure Operator storage needs:
  ```bash
  oc label node serius-master1 ran.openshift.io/lso=''
  oc label node serius-master2 ran.openshift.io/lso=''
  oc label node serius-master3 ran.openshift.io/lso=''
  ```

- Patch metal3 so it can see all the `bmh` resources in all namespaces:
    ```bash
    oc patch provisioning provisioning-configuration --type merge -p '{"spec":{"watchAllNamespaces": true}}'
    ```
## OpenShift GitOps configuration
- Prerequisite for this section: Be sure to download the argo-cd cli on the client system you're running the below commands on.
    ```bash
    wget https://github.com/argoproj/argo-cd/releases/download/v2.0.5/argocd-util-linux-amd64
    ```

- Configure a namespace called telco-gitops, a serviceaccount, clusterrole, and job that is necessary to function so that OpenShift Gitops can manage the cluster it is running on (management cluster).
    ```bash
    oc apply -k telco-gitops/
    ```

- To obtain the password for `openshift-gitops` ArgoCD `admin`.  Save this for later.

    ```bash
    oc get secret telco-gitops-cluster -n telco-gitops -o go-template='{{index .data "admin.password"}}' | base64 -d
    ```

### Definition of cluster for ArgoCD

After a cluster is deployed, before using ArgoCD for day-2 GitOps configurations, the cluster credentials must be defined in ArgoCD.

- Login into the ArgoCD instance in the management cluster using ArgoCD CLI. You will be prompted for the ArgoCD `admin` password.
    ```bash
    argocd login telco-gitops-server-telco-gitops.apps.ocp48-controlplane.vran.ibm.com --name admin
    WARNING: server certificate had error: x509: certificate is valid for telco-gitops, telco-gitops-grpc, telco-gitops.telco-gitops.svc.cluster.local, not telco-gitops-server-telco-gitops.apps.ocp48-controlplane.vran.ibm.com. Proceed insecurely (y/n)? y
    Username: admin
    Password:
    'admin:login' logged in successfully
    Context 'admin' updated
    ```

- List clusters
    ```bash
    $ argocd cluster list
    SERVER                                  NAME        VERSION  STATUS      MESSAGE
    https://kubernetes.default.svc          in-cluster  1.21     Successful
    ```

- Add target cluster
    ```bash
    $ argocd cluster add --kubeconfig ~/kubeconfig-serius admin --name serius
    INFO[0000] ServiceAccount "argocd-manager" created in namespace "kube-system"
    INFO[0000] ClusterRole "argocd-manager-role" created    
    INFO[0000] ClusterRoleBinding "argocd-manager-role-binding" created
    Cluster 'https://api.ocp48-controlplane.vran.ibm.com:6443' added
    ```

- Validate cluster has been defined
    ```bash
    argocd cluster list
    SERVER                          NAME        VERSION  STATUS      MESSAGE
    https://api.ocp48-controlplane.vran.ibm.com:6443  ocp48-controlplane        1.21     Successful  
    https://kubernetes.default.svc  in-cluster  1.21     Successful
    ```

- Go to the Telco GitOps Route (which should be exposed on the cluster) and log in as the admin user.
(https://telco-gitops-server-telco-gitops.apps.ocp48-controlplane.vran.ibm.com)

### ArgoCD view & testing

- Go into ArgoCD and notice there are multiple applications.  You will see 00-ocp48-controlplane-vran-base, and 00-ocp48-controlplane-config-assisted-installer.  Synchronize the Assisted Installer application to configure the AI operator running on the Management Cluster so it can deploy RAN clusters.  You can try to remove an Operator like OpenShift Serverless and see it be installed again, almost immediately by a reconciliation cycle from ArgoCD.

## Context / Background Output
This is output from a working management cluster (Serius) from September 2021 with versions at the time:

```bash
$ oc get installplan -A
NAMESPACE                 NAME            CSV                                            APPROVAL    APPROVED
assisted-installer        install-97sxb   assisted-service-operator.v99.0.0-unreleased   Automatic   true
openshift-local-storage   install-q4gtr   local-storage-operator.4.8.0-202109092008      Automatic   true
openshift-operators       install-2cg9n   hive-operator.v1.2.3179-0756b70e6              Automatic   true
openshift-operators       install-wxnvj   openshift-gitops-operator.v1.2.1               Automatic   true
openshift-serverless      install-9z42v   serverless-operator.v1.17.0                    Automatic   true
```
```bash
$ oc get operatorgroup -A
NAMESPACE                              NAME                           AGE
assisted-installer                     assisted-service-operator      21m
openshift-local-storage                openshift-local-storage        21m
openshift-monitoring                   openshift-cluster-monitoring   8d
openshift-operator-lifecycle-manager   olm-operators                  8d
openshift-operators                    global-operators               8d
openshift-serverless                   openshift-serverless           21m
```
