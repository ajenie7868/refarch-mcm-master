# Install Klusterlet on IBM Cloud Private cluster

For IBM Cloud Private(ICP) cluster to be managed by IBM Multicloud Manager (MCM), you need to deploy an MCM klusterlet on the ICP cluster. The procedure below describes on how to install the klusterlet. In this lab you will:
- Install klusterlet on ICP cluster

## Prerequisites
<!-- - MCM installable ( ex: mcm-3.1.1.tgz ) -->
Three environments are needed for this lab:
1. An operational MCM Hub Cluster
    - HTTP address: https://master1.ibmserviceengage.com:8443/
    - User Name: your LDAP user ID. E.g. `user01`
    - Password: `passw0rd`

2.	A dedicated managed cluster
    - HTTP address: will be shared by the instructor
    - User Name: `admin`
    - Password: `admin_4890`

3. A workstation configured with the ICP CLIs:
    - [cloudctl CLI](cloudctl.md)
    - [kubectl CLI](kubectl.md)
    - [Helm CLI](helm.md)

    If you don't have the CLIs installed, you can download and install them from your dedicated managed cluster, at https://<MANAGED_CLUSTER_IP>:8443/console/tools/cli

## Get token from MCM hub cluster
- Log in to the MCM hub cluster console: https://master1.ibmserviceengage.com:8443
- Click on user Avatar and select Configure Client

![alt text](https://github.ibm.com/CASE/refarch-mcm/blob/master/labs/Lab%201%20-%20MCM%20set-up/UserAvatar.png)
- Copy and paste the entire text in this box in a text editor

![alt text](https://github.ibm.com/CASE/refarch-mcm/blob/master/labs/Lab%201%20-%20MCM%20set-up/kubectl_config.png)

- Copy the token, after the clause `token=`. E.g.:  
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdF9oYXNoIjoiaXppNjl6NzlnOG52ZDk5OHB3cXUiLCJyZWFsbU5hbWUiOiJjdXN0b21SZWFsbSIsInVuaXF1ZVNlY3VyaXR5TmFtZSI6ImFkbWluIiwiaXNzIjoiaHR0cHM6Ly9tYXN0ZXIxLmlibXNlcnZpY2VlbmdhZ2UuY29tOjk0NDMvb2lkYy9lbmRwb2ludC9PUCIsImF1ZCI6ImE0NDNiN2Y4ZjkzNzNjMDRhMTYzOTc3MDhiNTUyOWJhIiwiZXhwIjoxNTQ2NzI1NDg5LCJpYXQiOjE1NDY2OTY2ODksInN1YiI6ImFkbWluIiwidGVhbVJvbGVNYXBwaW5ncyI6W119.VPHiGNbJ1O8J7H7EKqBuYZD0wp7Res2BTVdlNRwSiBDTJl7jyj3archWtjZFaF2u5BJjy-1Nx9IaOPTUx2VIRrnuvNgLVT9FtD0tP3chOk0WmiSLNjEapf49VkTl1KHlrrnU1rOwQwn60c6frWxB_oPBeAD0cgxz6Jl5jD-hmZ--ju1mmY6OppK-9vnq_OF3pEfLOJZQVfo7dLzo7p8QpTxPkpunfUWeOqr1IKBX9PAvylrXGkHAeNpZP0SAnn3ISBlSa2L1RftJ6HnP2mzENkDQFLzxDjXb6OtPPcWq7yJ68LRw07PxpzT7GGVHr4NnyxFwTv4Tv_essGmCtNqnA
```

**Note: Make sure you copy the entire token, not just what's visible in the browser without scrolling right.**

## Install Klusterlet on the managed cluster
The MCM package which contains docker images and helm charts need to be loaded into the ICP cluster where the MCM klusterlet is going to be installed. The target cluster name will be in the format `mcm-userxx-cluster` where `xx` is unique to the user. For example `user01` will have the clustername as `mcm-user01-cluster`
- Log into the target ICP cluster using cli `cloudctl`. Please replace `https://10.0.0.1:8443` with the url shared by instructor which is unique to each user. E.g.:
```
cloudctl login -a https://10.0.0.1:8443 --skip-ssl-validation -u admin -p admin_4890 -n kube-system
```
<!--
- Log into the docker registry. Please replace `user01` with your `userid`. E.g.: for user01.
```
docker login mcm-user01-cluster.icp:8500 -u admin -p admin_4890
```
-->
### Create Tiller Secret
- Create a secret for tiller. Depending on the *HELM_HOME* variable ( can be `~/.helm` or `/var/lib/helm` ), the command to create the secret is
```
kubectl create secret tls klusterlet-tiller-secret --cert ~/.helm/cert.pem --key ~/.helm/key.pem -n kube-system
```
 or
```
kubectl create secret tls klusterlet-tiller-secret --cert /var/lib/helm/cert.pem --key /var/lib/helm/key.pem -n kube-system
```
- You will get the following result:
```
secret/klusterlet-tiller-secret created
```
<!--
### Upload MCM package
- Untar the MCM installable ( mcm-3.1.1.tgz )
```
tar zxf mcm-3.1.1.tgz
cd mcm-3.1.1
```
- Load the package mcm-3.1.1-amd64.tgz into local docker registry and helm repository. Have to specify the docker registry. Please replace `user01` with your `userid`.
```
cloudctl catalog load-archive --archive ./mcm-3.1.1-amd64.tgz --registry mcm-user01-cluster.icp:8500/kube-system
```
-->

### Patch Cluster Image Policy
As of ICP 3.1, images can be pulled only from authorised docker registries. The following command allows for images from **any** docker registry to be deployed to your ICP cluster.

**Do not run the following command in any system apart from demo ones.**
- Run the following command:
```
REPO="*"; kubectl patch clusterimagepolicies $(kubectl get clusterimagepolicies --no-headers | awk '{print $1}') -p '[{"op":"add","path":"/spec/repositories/-","value":{"name":"'${REPO}'"}}]' --type=json
```
- You will get the following result:
```
clusterimagepolicy.securityenforcement.admission.cloud.ibm.com/ibmcloud-default-cluster-image-policy patched
```

### Deploy MCM Klusterlet Helm Chart
- Log in to the target ICP console. The URL is shared by instructor with username `admin` using password `admin_4890`
- Sync the helm repositories, Menu -> Manage – Helm Repositories and then click on Sync Repositories
- Wait 30 seconds
- Click on the catalog ( top right )
- In the filter, search for mcm

![alt text](https://github.ibm.com/CASE/refarch-mcm/blob/master/labs/Lab%201%20-%20MCM%20set-up/Catalog.png)

- Click on **ibm-mcmk-prod** in the service catalog and then click on Configure. The table below shows the example values that need to be filled. For the rest of the fields, please leave the default values. Expand All Parameters to show all the fields. Please replace `user01` with your `userid`.

| Field	| Value |
| ----- | ----- |
| Helm Release Name | mcmklusterlet |
| Target namespace | kube-system |
| Accept the license | check the box |
| Cluster Name | `mcm-user01-cluster` |
| Cluster Namespace | `mcm-user01-ns` |
| Hub Cluster Kubernetes API Server | https://master1.ibmserviceengage.com:8001 |
| Hub Cluster Kubernetes API Server token | **Value of token you copied from MCM Hub Cluster** |
| Klusterlet Tiller Secret Name | klusterlet-tiller-secret ( created earlier by you ) |
| Enable Automatic Generate Tiller Secret | **Untick** this box, as you have already generated the secret |
| Cluster Cloud Provider | IBM |
| Kubernetes Vendor | ICP |
| Cluster Environment Type | Prod |
| Cluster Region | US |
| Cluster Datacenter | dallas |
| Cluster Owner | *Your LDAP User ID e.g. `user01`* |

- Click on Install and then check the helm release to check the status of the klusterlet resources deployment.
- Validate the status of the pods via the cloudctl CLI:
```
kubectl get pods --all-namespaces | grep mcm
mcmklusterlet-ibm-mcmk-prod-klusterlet-5f868688bd-j8d79        4/4       Running     0          2m
mcmklusterlet-ibm-mcmk-prod-weave-scope-7clk4                  1/1       Running     0          2m
mcmklusterlet-ibm-mcmk-prod-weave-scope-app-574c64f98d-lm5wg   2/2       Running     0          2m
mcmklusterlet-ibm-mcmk-prod-weave-scope-kkzpt                  1/1       Running     0          2m
```
- Log in to the hub cluster in a browser with url `https://master1.ibmserviceengage.com:8443` using userid `userxx` with password `passw0rd`. Please replace `userxx` with your LDAP user ID, E.g. `user01`.
- From the hamburger menu, click on `Multicloud Manager`, then `Clusters`, and check that the table shows your ICP cluster.
