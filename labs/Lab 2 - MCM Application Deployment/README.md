# MCM Application Deployment

In this Lab, you will:
- package an application CRD
- deploy the application via Helm
- change the placement policy to move the application to your managed cluster

## Prerequisites
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
    - cloudctl CLI
    - kubectl CLI
    - Helm CLI

    If you don't have the CLIs installed, you can download and install them from your dedicated managed cluster, at https://<MANAGED_CLUSTER_IP>:8443/console/tools/cli

## Guestbook
The application used in this lab is a PHP and Redis app based on this [example](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/).

It has been refactored into an MCM application CRD, as per this [example](https://github.ibm.com/IBMPrivateCloud/hybrid-cluster-manager-v2-chart/tree/master/guestbookv2).

An MCM application model includes:
- Deployables
- Relationships
- Placement Policies

Each deployable can be deployed via its Helm chart.
In this example, also the application CRD itself will be packaged into a Helm chart.

For more information on the MCM application model, please refer to https://github.ibm.com/IBMPrivateCloud/roadmap/tree/master/feature-specs/hcm/application-model

### Explore Guestbook
Browse to `guestbook/gbapp/templates` to review the application's components.
- The `application.yaml` file defines the application.
- The different *deployables* specify the application components, and point at the Helm chart deploying that component. In this case, the Helm charts are stored in a public GitHub repository.
- The *relationship* files define the relationships between components, specifying a `source` and a `destination`.
- The `placement.yaml` file defines the placement policy used by MCM to place the application. The application will only be deployed to clusters federated to the MCM controller which have labels corresponding to the ones specified in this file.
- The `placementbinding.yaml` file introduces fine-grained placement policies for specific application components, rather than the whole application. This file is not used in MCM versions prior to 3.1.2

### Package the application
- Clone the top level of this [github repo](https://github.ibm.com/CASE/refarch-mcm) and copy the `labs/Lab 2 - MCM Application Deployment/guestbook` folder to your file system
- You can review the code for the gbapp helm chart in `guestbook/gbapp`
- Package the the Application CRD into a Helm chart:
  ```
  helm package guestbook/gbapp
  ```
- Make sure that the file `gbapp-0.1.0.tgz` has been generated.

**For Fast Start Lab 2, this chart has already be uploaded to the hub cluster, and you don't need to upload it again.**
<!--
- Upload the Helm chart to the hub cluster repository.
```
cloudctl catalog load-chart --archive gbapp-0.1.0.tgz
```
-->

## Deploy Guestbook
Guestbook is now packaged as a Helm chart and uploaded to the ICP catalogue on the hub cluster.
From here, the Helm chart with application wrapper (`application.yaml`) can be deployed locally, and the placement policy will in turn determine to which cluster the application components will be deployed.
The application wrapper can be deployed from the catalog UI or via the CLI.

**Note: as of ICP 3.1 the clusterimagepolicy needs to edited to allow the guestbook images to be used. In the VM used in this lab, clusterimagepolicy has already been edited to allow the deployment of guestbook images to the hub and all the managed clusters (as you would have done in Lab 1).**

### Deploy the MCM application via UI
- Browse the ICP catalog on the hub cluster: https://master1.ibmserviceengage.com:8443/catalog/
- Select for the chart called `gbapp`, then press `Configure`
- Give the Helm release a name composed by the prefix `gb-` and your LDAP user ID. E.g. `gb-user01`
- Select the namespace composed by the prefix `mcm-` followed by your LDAP user ID and the suffix `-ns`. E.g. `mcm-user01-ns`. This will be the `TARGET_NAMESPACE` for the application wrapper.
- Expand the `All parameters` tab to fill out the parameters which will be used for the placement policy:
  - `replicaCount` specifies the number of replicas of the deployed applications across the clusters (maximum one application per cluster is deployed). Set this parameter to `1`
  - `appInClusterNamespace` specifies the namespace to which the application components will be deployed.

    **Do not leave this parameter blank, or the deployment will fail.**

    In `appInClusterNamespace`, type a namespace composed by the prefix `mcm-` followed by your LDAP user ID and the suffix `-ns`. E.g. `mcm-user01-ns`.

    Although the `TARGET_NAMESPACE` for the application wrapper and and `appInClusterNamespace` could be different, in this lab you are going to use the same namespace.

    Make a note of the namespace you have passed in `TARGET_NAMESPACE` and `appInClusterNamespace`.
  - the `targetCluster` labels specify cluster labels used by MCM to determine where to deploy the application components. In this lab, the labels have been pre-populated via the `values.yaml` file, in order to deploy the application chart to the hub cluster. Leave the labels as they are.
- Click on `Local Install`. This will deploy the application wrapper to the hub cluster.

## Review Guestbook deployment

### Access Guestbook
Apart from the application wrapper, 3 additional Helm releases will be deployed to the hub cluster, per the placement labels specified, each of those with a single K8s deployment: Guestbook's front end, redis master, and redis slave.
- In the hub cluster ICP UI (https://master1.ibmserviceengage.com:8443/), go to `Workloads` > `Deployments`
- If you get a `You are unauthorized to view this page.` message, make sure you select the `appInClusterNamespace` name in the namespace filter in the top-right corner of your screen.
![alt text](https://github.ibm.com/CASE/refarch-mcm/blob/master/labs/Lab%202%20-%20MCM%20Application%20Deployment/namespace_filter.png)
- Find the deployment which contains the Guestbook front end: `md-[HELM_RELEASE_NAME]-gbapp-hub-cluster-gbf`. E.g. `md-gb-user01-gbapp-hub-cluster-gbf`
- Click on the `Launch` hyperlink to open the Guestbook front end
- You can save names into the Guestbook, if you like.

### Review the MCM application deployment
- In the hub cluster ICP UI (https://master1.ibmserviceengage.com:8443/), open the `Multicloud Manager` view, from the hamburger menu on the top right corner of the page.
- Click on `Clusters` to observe the managed clusters federated to the hub, with their labels.
- Click on `Applications` to review the list of deployed MCM applications.
- Identify the app you that you have just deployed, e.g. `gb-user01-gbapp`, and on click on the `Launch Health View` hyperlink on its right. This will open a set of Grafana dashboards which get dynamically generated by MCM when an application is deployed.
- **Leave the Grafana tab open**, and go back to the `Multicloud Manager` > `Applications` tab in your browser.
- Click on the application that you have just deployed, E.g. `gb-user01-gbapp`.
- The *Overview* tab lists the application components and labels used by the placement policy.
- The *Design* tab provides a graphical representation of the application components and their relationships, as described in the relationship files that we have mentioned earlier.
  - Clicking on the single components highlights the yaml definition of that component.
- The *Topology* tab monitors where the components have been deployed by MCM. It is pre-configured to look for labels defining the application. E.g. `app:gbf`.
  - In the *Clusters* filter, change the value to select: `datacenter: dallas` and `datacenter: rochester`.
  - In the *Namespaces* filter, change the value to select the namespace you have passed in `appInClusterNamespace`. E.g. `mcm-user01-ns`.
  - In the *Types* filter, change the value to select: `deployment`, `pod`, and `service`.
  - You should now be able to view the components deployed via your helm chart, as well as their interactions.
  - **Keep this tab open in your browser**

## Change placement policy to move the application
When deploying the application Helm chart, you   specified labels to create the placement policy for the guestbook application deployment. These labels can be edited at any point, either manually or programmatically, to influence the application placement across the clusters federated into the MCM controller.

In this section of the lab, you will manually change the placement policy to move your application deployment from the hub cluster to your dedicated managed cluster.

Before doing that, a CustomRoleBinding (CRB) needs to be created on the managed cluster, in order to allow for the deployment of the guestbook components.

This can be achieved in different ways:
- Manually
- Through a compliance policy (not explored in this lab)
- Through placement binding (not available in MCM 3.1.1)

### Create ClusterRoleBinding on the managed cluster
In this lab, we will create the ClusterRoleBinding (CRB) manually:
- Make a copy of the `crb-template.yaml` [file](https://github.ibm.com/CASE/refarch-mcm/blob/master/labs/Lab%202%20-%20MCM%20Application%20Deployment/crb-template.yaml) in [this repo](https://github.ibm.com/CASE/refarch-mcm/), by either copying and pasting its content to a file, or use the file attained through the repo clone.
- Edit your copy of `crb-template.yaml`, filling out the following parameters:
  - `[CRB_NAME]` is the name of your CRB. Use your LDAP user ID followed by the suffix `-crb`. E.g. `user01-crb`
  - `[TARGET_NAMESPACE]` is the namespace where you'll have the right to deploy. It should be the same namespace that you have passed in `appInClusterNamespace` when deploying the Guestbook Helm chart. E.g. `mcm-user01-ns`
- Log in using the cloudctl CLI to the **managed cluster**. Please replace `MANAGED_CLUSTER_IP` with the url shared by instructor which is unique to each user:
  ```
  $ cloudctl login -a [MANAGED_CLUSTER_IP] --skip-ssl-validation -u admin -p admin_4890 -n kube-system
  ```
- Apply the ClusterRoleBinding:
  ```
  $ kubectl apply -f [FILE_LOCATION]/crb-template.yaml
  ```
  Where `FILE_LOCATION` is the location where you have saved your local copy of the `crb-template.yaml` file.

### Review the placement policy
- Log in using the cloudctl CLI to the **hub cluster**:
  ```
  $ cloudctl login -a https://master1.ibmserviceengage.com:8443/ --skip-ssl-validation -u [YOUR_LDAP_USER_ID] -p passw0rd -n [TARGET_NAMESPACE]
  ```
  Where `YOUR_LDAP_USER_ID` is your LDAP user ID (e.g. `user01`), and `TARGET_NAMESPACE` is the namespace that you have passed in when deploying the Guestbook Helm chart (e.g. `mcm-user01-ns`).
- Retrieve the placement policy for your application:
  ```
  $ kubectl get pp -n [TARGET_NAMESPACE]
  ```
- Observe that the *decision* made by MCM is to deploy the Guestbook app to the hub cluster.

### Edit the placement policy
- Edit the placement policy manually:
  ```
  $ kubectl edit pp <placement_policy_name> -n [TARGET_NAMESPACE]
  ```
Where `<placement_policy_name>` is the name of the policy you have just retrieved, and `[TARGET_NAMESPACE]` is the namespace where the policy has been deployed.
- Using *vi* commands:
  - Remove the following lines from the cluster selector spec:
    - `datacenter: rochester`
    - `environment: Dev`
  - Edit the `owner: offering` line to read:
  ```
  owner: [CLUSTER_OWNER]
  ```
  Where `CLUSTER_OWNER` is the label you specified when you deployed the klusterlet in Lab 1, and should correspond to `YOUR_LDAP_USER_ID`, e.g. `user01`.

  If you don't remember which label you specified as the `Cluster Owner` of your dedicated manged cluster, you can find it in the `Multicloud Manager` UI > `Clusters`.
- Observe that the number of replicas is 1. We will not change this value, as we want to move the application from one cluster to another, rather than scaling it out.
- Save the placement policy by pressing `:x` in the CLI editor

### Review the application placement after the policy change
- Retrieve the placement policy for your application:
  ```
  $ kubectl get pp -n [TARGET_NAMESPACE]
  ```
  Where `TARGET_NAMESPACE` is the namespace that you have passed in when deploying the Guestbook Helm chart. E.g. `mcm-user01-ns`
- Observe that the *decision* made by MCM is now to deploy the Guestbook app to your managed cluster. MCM has evaluated the labels and made a new decision.
- Go back to `Multicloud Manager` UI > `Applications` > Your application > `Topology` (you should have this tab still open in your browser), to observe the following:
  - The pods and services on the hub clusters being killed
  - New pods being created on your dedicated managed cluster

![alt text](https://github.ibm.com/CASE/refarch-mcm/blob/master/labs/Lab%202%20-%20MCM%20Application%20Deployment/Application_Movement.png)

- If you have the Grafana tab still open, refresh the browser page, and notice how the dashboards get dynamically updated to monitor components on your managed cluster, rather than the hub cluster.
- From the managed cluster UI > `Workloads` > `Deployments `, you will be able to launch the new guestbook front end.
