# IBM Cloud Pak for Data Installation Spark Notes

## Installation Notes
1. Login with OpenShift cli
```shell
oc login 
```
2. Clone repo to the client workstation or download as a zip from git.
```shell
git clone -b v2 https://github.com/ekleinso/cpd-alt-install.git
```
3. Change into directory ***cpd-alt-install***.
```shell
cd cpd-alt-install
```
4. Make sure the 4 projects are created in OpenShift

| Cloud Pak Service | Project Name |
| :---------------------- | :------------ |
| PROJECT_LICENSE_SERVICE | ibm-licensing |
| PROJECT_SCHEDULING_SERVICE | ibm-scheduler |
| PROJECT_CPD_INST_OPERATORS | ibm-operators |
| PROJECT_CPD_INST_OPERANDS | ibm-instance |

```shell
export PROJECT_LICENSE_SERVICE="ibm-licensing"
export PROJECT_SCHEDULING_SERVICE="ibm-scheduler"
export PROJECT_CPD_INST_OPERATORS="ibm-operators"
export PROJECT_CPD_INST_OPERANDS="ibm-instance"
```
5. Create service account in **PROJECT_CPD_INST_OPERANDS**
```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f service-account.yaml
```
6. Update variables in **configmap.yaml** for your environment
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cpd-vars
data:
  PROJECT_LICENSE_SERVICE: "ibm-licensing"
  PROJECT_SCHEDULING_SERVICE: "ibm-scheduling"
  PROJECT_CPD_INST_OPERATORS: "ibm-operators"
  PROJECT_CPD_INST_OPERANDS: "ibm-instance"
  OPENSHIFT_TYPE: "self-managed"
  IBM_ENTITLEMENT_KEY: "<your entitlement key>"
  COMPONENTS: "analyticsengine"
  VERSION: "5.2.0"
  IMAGE_ARCH: "amd64"
  STG_CLASS_BLOCK: "sc-ontap-nas"
  STG_CLASS_FILE: "sc-ontap-nas"
  OCP_URL: "kubernetes.default.svc.cluster.local"
  OLM_UTILS_IMAGE: "registry.example.org/docker/cpopen/cpd/olm-utils-v3:latest"
```

```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f configmap.yaml
```
7. Update **subjects[0].namespace** in **rolebindings.yaml** if you changed the ***PROJECT_CPD_INST_OPERANDS*** variable in **configmap.yaml**
```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ibmce-cpd-deployment
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: ibmce-deployment-sa
    namespace: ibm-instance
```

```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f rolebindings.yaml
```
8. Update value for ***storageClassName*** in all of the entries in **storage.yaml** if you changed the ***STG_CLASS_FILE*** variable in **configmap.yaml** before you run command to create storage.
```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f storage.yaml
```
9. Create secrets
```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f secret.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-00.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-01.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-02.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-03.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-04.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-05.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-06.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-07.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-08.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-09.yaml
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f case-10.yaml
```
10. Update ***spec.containers[0].image*** in **1-pod-shared.yaml** to point to the correct repository/image as necessary for your environment. Create pod to invoke CPD install
```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f 1-pod-shared.yaml
```
11. Monitor install log and/or check pod status until it is completed
```shell
oc logs -n ${PROJECT_CPD_INST_OPERANDS} -f -l app=cpd-shared
```
or 
```shell
oc -n ${PROJECT_CPD_INST_OPERANDS} get po -l app=cpd-shared
```
12. Update ***spec.containers[0].image*** in **2-pod-cpd.yaml** to point to the correct repository/image as necessary for your environment. Create pod to invoke CPD install
```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f 2-pod-cpd.yaml
```
13. Monitor install log and/or check pod status until it is completed
```shell
oc logs -n ${PROJECT_CPD_INST_OPERANDS} -f -l app=cpd-install
```
or 
```shell
oc -n ${PROJECT_CPD_INST_OPERANDS} get po -l app=cpd-install
```
14. Update ***spec.containers[0].image*** in **3-pod-services.yaml** to point to the correct repository/image as necessary for your environment. Create pod to invoke CPD install
```shell
oc create -n ${PROJECT_CPD_INST_OPERANDS} -f 3-pod-services.yaml
```
15. Monitor install log and/or check pod status until it is completed
```shell
oc logs -n ${PROJECT_CPD_INST_OPERANDS} -f -l app=cpd-services
```
or 
```shell
oc -n ${PROJECT_CPD_INST_OPERANDS} get po -l app=cpd-services
```
