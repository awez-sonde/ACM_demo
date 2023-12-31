# Odido RHACM Workshop 


## Access the console 

Once logged in, navigate to the MultiCluster Switcher and select All Clusters to see the ACM UI.


Now, navigate to the Clusters screen, you will notice that the Hub cluster is already in the list as a managed cluster (local-cluster), we will use this later in the guide.



## Cluster Lifecycle (Cluster Lifecycle Management use case)

At a high level Cluster Lifecycle management is about creating, upgrading, and destroying and importing clusters in a multi cloud environment.

Its time to use your  AWS credentials, Access Key ID, Secret Access Key, and the Base DNS Domain.  In order to create a new OpenShift cluster in the AWS cloud we will need these keys to create a Provider Connection. On the left bar, select Credentials and then select Add Credential.

You will need to provide connection details
```
 Credential Type: Choose Amazon Web Services and then, Amazon Web Services again
 Credential Name:  aws
 Namespace: open-cluster-management
 Base DNS Domain:  your_base_dn
```
 - Click NEXT

```
 Access Key ID:  your-access-id
 Secret Access Key ID: your-secret-id
```

- Click NEXT - We don’t need to configure a Proxy
```
 Red Hat OpenShift pull secret:  Get from your [Red Hat login](https://cloud.redhat.com/openshift/install/pull-secret)
 SSH private and public keys:  Use an existing key pair or generate a new one (see link below)
```
- Click NEXT
- Verify the information and click ADD


Please refer to [Creating a cloud connection for Amazon Web Services](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.3/html/credentials/credentials#aws_cred_create)   for more information on how to complete the step.


### Create a new OpenShift cluster in AWS

- From the Clusters page, select Create Cluster
- Select Amazon Web services and then Standalone.
- Select the Infrastructure provider credential: aws (may be already selected)
- Name: cluster1 
- Leave the Cluster set empty for now 
- Select a Release Image, chose a 4.13 version
- Add a label of environment=prod. Click NEXT
- Change the region to *see table below*

---
**NOTE**

Please note that the deployment might fail, if so please retry at different availability zone, we have very limited amount of elastic IP addresses in these accounts.

---

Leave everything else as is(default) and then click NEXT on the other screens or select 7 - Review and create on the menu and then click CREATE 

In about 45 minutes this new cluster will be ready to go!  

If you choose an earlier version of OpenShift, you will also have the option to upgrade OpenShift if you like.  This will also take some time.


## Application Lifecycle (Application Lifecycle management use case)


In the previous lab, you explored the Cluster Lifecycle functionality in RHACM. This allowed you to register/import an OpenShift® cluster to RHACM, which you can now use to deploy applications.

Application Lifecycle functionality in RHACM provides the processes that are used to manage application resources on your managed clusters. This allows you to define a single or multi-cluster application using Kubernetes specifications, but with additional automation of the deployment and lifecycle management of resources to individual clusters. An application designed to run on a single cluster is straightforward and something you ought to be familiar with from working with OpenShift fundamentals. A multi-cluster application allows you to orchestrate the deployment of these same resources to multiple clusters, based on a set of rules you define for which clusters run the application components.


The table below describes the different components that the Application Lifecycle model in RHACM is composed of:



| Resource | Purpose |
| ---- | ---- |
| Channel | Defines a place where deployable resources are stored, such as an object store, Kubernetes namespace, Helm repository, or GitHub repository |
| Subscription | Definitions that identify deployable resources available in a Channel resource that are to be deployed to a target cluster |
| Placement | Defines the target clusters where subscriptions deploy and maintain the application. It is composed of Kubernetes resources identified by the Subscription resource and pulled from the location defined in the Channel resource. |
| Application | A way to group the components here into a more easily viewable single resource. An Application resource typically references a Subscription resource. |

These are all Kubernetes custom resources, defined by a Custom Resource Definition (CRD), that are created for you when RHACM is installed. By creating these as Kubernetes native objects, you can interact with them the same way you would with a Pod. For instance, running oc get application retrieves a list of deployed RHACM applications just as oc get pods retrieves a list of deployed Pods.


This may seem like a lot of extra resources to manage in addition to the deployables that actually make up your application. However, they make it possible to automate the composition, placement, and overall control of your applications when you are deploying to many clusters. With a single cluster, it is easy to log in and run oc create -f…​. If you need to do that on a dozen clusters, you want to make sure you do not make a mistake or miss a cluster, and you need a way to schedule and orchestrate updates to your applications. Leveraging the Application Lifecycle Builder in RHACM allows you to easily manage multi-cluster applications.


### Creating an Application

#### Prerequisites:  
- On the local cluster add a label:  environment=dev
- On the new cluster you provisioned via ACM double check you added the label:  environment=prod

#### Deployment
- In RHACM, navigate to Applications click Create application and select Subscription. 
- Enter the following information:
  Next to Create application, make sure the YAML dial is ON
```
Name: book-import
Namespace: book-import 
```

- Under Repository location for resources, select the GIT repository
```
URL:  https://github.com/hichammourad/book-import.git
Branch:  master-no-pre-post
Path:  book-import
```

- Under the Select clusters for application deployment, select Deploy application resources on clusters with all specified labels
```
Cluster sets: global
Label: environment
Value: dev
```


- Click Create and after a few minutes you will see the application and all its components available in RHACM.

If everything was done correctly you should be able to see the application deployed to local-cluster. Go to Applications, and make sure to filter by subscription 

This will show only the apps deployed from ACM, instead of all the existing apps in the managed clusters. 

- Click on the book-import application and have a look at the topology view

- Select the Route and click on the URL provided, you should see the Book Import application

- See the Book Import user interface.

Feel free to experiment with the application.  Edit it and change the label to environment=prod.  What happens to the application?

You have now completed the overview of the Application Lifecycle functionality in RHACM.

You successfully deployed an application to a target cluster using RHACM. This approach leveraged a Git repository which housed all of the manifests that defined your application. RHACM was able to take those manifests and use them as deployables, which were then deployed to the target cluster.

You also leverage the power of labels and deploy the application to your imported cluster. I highly encourage you to play around with the labels and deploy this application to your local cluster. You can also create other clusters and or applications if you so desire.



## Governance, Risk, and Compliance (Security and compliance use case)


### Creating Policies in ACM

At this point, you have completed the overview labs for Cluster Lifecycle and Application Lifecycle capabilities in RHACM. In the Cluster Lifecycle Lab, you learned how RHACM can help manage the lifecycles of your Kubernetes clusters, including both deploying new clusters and importing existing clusters. In that lab, you configured your RHACM instance to manage an OpenShift® cluster.

In the Application Lifecycle Lab, you continued exploring RHACM functionality and learned how to deploy and configure an application. You used the cluster that you added in the first module as the target for deploying an application.

Now that you have a cluster and a deployed application, you need to make sure that they do not drift from their original configurations. This kind of drift is a serious problem because it can happen from beginning, and benevolent fixes and changes, as well as malicious activities that you might not notice but can cause significant problems. The solution that RHACM provides for this is the Governance, Risk, and Compliance, or GRC, functionality.

### Review GRC Functionality

To begin, it is important to define exactly what GRC is. In RHACM, you build policies that are applied to managed clusters. These policies can do different things, which are described below, but they ultimately serve to govern the configurations of your clusters. This governance over your cluster configurations reduces risk and ensures compliance with standards defined by stakeholders, which can include security teams and operations teams

This table describes the three types of policy controllers available in RHACM along with the remediation mode they support:

| Policy Controller | Purpose | Enforce/Inform |
| ----- | ----- | ----- |
| Configuration | Used to configure any Kubernetes resource across your clusters. Where these resources are created or configured is determined by the namespaces you include (or exclude) in the policy | Both |
| Certificate | Used to detect certificates that are close to expiring. You can configure the certificate policy controller by updating the minimum duration parameter in your controller policy. When a certificate expires in less than the minimum duration, the policy becomes non-compliant. Certificates are identified from secrets in the included namespaces. | Inform |
| Identity and Access Management (IAM) | The IAM policy controller monitors for the desired maximum number of users with a particular cluster role (i.e. ClusterRole) in your cluster. | Inform |

You need to create three different resources in order to implement the policy controllers:

| Resource | Function |
| ----- | ----- |
| Policy | The Policy defines what you actually want to check and possibly configure (with enforce). Policies include a policy-template that defines a list of objectDefinitions. The policy also determines the namespaces it is applied to, as well as the remediation actions it takes. |
| PlacementRule | Identifies a list of managed clusters that are targeted when using this PlacementRule. |
| Placement Binding | Connect the policy to the PlacementRule. |


We’ll go through a simple example, and create a policy in the default namespace.
For this, we’ll need a little setup:

- Navigate to Clusters and access the ClusterSets tab.
- Click on the 3-dots button on the right on the global cluster set, and then on Edit namespaces bindings and add the default namespace.
- Navigate to the Governance screen and click create policy.
- Build a policy with the following information:
  ```
  Name: policy-grc-cert
  Namespace: default
  ```
- Click NEXT.
- Policy Templates: Click on Add policy template and select Certificate Management expiration.
- Click NEXT.
- On Placement, click on New Placement.
- Select global as the clusterSet.
- Leave everything else as default and click NEXT.

As you can see on the Review page, this policy will be applied to every cluster in the global ClusterSet, and it will look for expired certificates in them. If there’s a certificate that is set to expire in less time than the minimumDuration specification, the policy will inform a non-compliant status.

- Click SUBMIT.

Please note that initially, it will complain that there is an issue with the policy but shortly after should go green and get a checkmark.

You can also play around by creating more policies if needed, we have some examples in the [Collection of policy examples for Open Cluster Management](https://github.com/open-cluster-management/policy-collection)


### Bonus exercise

Lets monitor openshift-authentication certificate 

- Extract the authentication serving cert and check the validity using the below command from the Bastion host
  ```
  oc extract secret/v4-0-config-system-serving-cert -n openshift-authentication --confirm && openssl x509 -in tls.crt -noout -text  | grep -iE "not after"
  ```
- Calculate the number of hours till the date that we got in the previous step.You can do it from [here](https://www.timeanddate.com/date/duration.html?y1=2023&m1=11&d1=11&y2=2025&m2=11&d2=9)

- Use a higher number as a MinimumDuration in our policy templates.Dont forget to add openshift-authentication in the clusterset binding and in the policy templates.


## End to End Visibility (Observability use case)

View system alerts, critical application metrics, and overall system health. Search, identify, and resolve issues that are impacting distributed workloads using an operational dashboard designed for Site Reliability Engineers (SREs).  This is done via the integration of Grafana.  Let's walk through the steps to integrate Grafana with ACM.

You will need your AWS Keys. 
You will also need to create an AWS S3 bucket.
SSH information to your bastion host. 

### Create the S3 bucket
- Login to the bastion host.
- Run the following command to log in to AWS:
  ```
   aws configure  
  ```
  and enter your AWS keys when prompted.
  ```
  Default region: us-east-2
  Default output format : [None}
  ```
- Then run the following command to create the S3 bucket:
  ```
   aws s3 mb s3://grafana-$GUID
  ```
- Please take note of the bucket name.


### Integrate Grafana into ACM

- Login to the bastion host host.
- Create a namespace by running the following command: 
```
oc create namespace open-cluster-management-observability
```
- Copy the pull secret into this new namespace by running the following TWO commands:  
```
DOCKER_CONFIG_JSON=`oc extract secret/pull-secret -n openshift-config --to=-` 



oc create secret generic multiclusterhub-operator-pull-secret -n open-cluster-management-observability --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" -- type=kubernetes.io/dockerconfigjson
```
- In your current folder create a file called thanos-object-storage.yaml  and add the following text in the file. Please be sure to update your S3 bucket name and AWS Keys

```
apiVersion: v1
kind: Secret
metadata:
  name: thanos-object-storage
type: Opaque
stringData:
  thanos.yaml: |
    type: s3
    config:
      bucket: YOUR_S3_BUCKET
      endpoint: s3.amazonaws.com
      insecure: false
      access_key: YOUR_ACCESS_KEY
      secret_key: YOUR_SECRET_KEY

```

- Create a secret for your object storage by running the following command:
```
oc create -f thanos-object-storage.yaml -n open-cluster-management-observability
```
- Create the MultiClusterObservability custom resource for your managed clusters.  To do this create a  YAML file named multiclusterobservability_cr.yaml

```
kind: MultiClusterObservability
apiVersion: observability.open-cluster-management.io/v1beta2
metadata:
  name: observability
spec:
  observabilityAddonSpec: {}
  storageConfig:
    metricObjectStorage:
      key: thanos.yaml
      name: thanos-object-storage
```
---

**Note**: This is the default operation. There are multiple other optional fields to customize this resource. If you want to change optional parameters like retentionResolution, storageClass to use, etc., please check the API reference.

---


- Apply the observability YAML to your cluster by running the following command:
  ```
  oc apply -f multiclusterobservability_cr.yaml
  ```
- Wait for the observability pods to start
  ```
  watch oc get pods -n open-cluster-management-observability
  ```
- Once all the pods are up log in to the ACM console, and Infrastructure 
- Click on the Grafana link in the top right to view the metrics from the managed clusters.  Please note: it will take a few minutes for the metrics to become visible on the dashboard.
- On the right top corner explore the Dashboards on Grafana.

Would you like to do more with Grafana?  Please visit [this page](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.7/html/observability/index)



## RHACM and GitOps
You can integrate the OpenShift GitOps Operator with Advanced Cluster Management for Kubernetes which provides advanced features compared to the Application and Subscription model offered by RHACM itself.
OpenShift GitOps is the downstream project for ArgoCD, and it benefits all the latest features and enhancements whilst receiving support from the OpenShift GitOps subscription.
### Installing GitOps Operator
As a prerequiste, you must install the GitOps Operator on the Hub Cluster, so head to the Local Cluster console, in the Operators->OperatorHub section, and search for "Red Hat OpenShift GitOps". Click on "Install", leave everything as default and click again on "Install", and wait for the installation to finish.
Once installed, we will have our ArgoCD instance created in the "openshift-gitops" namespace.
### Configuring Managed Cluster into GitOps
Next we need to create the necessary resources to enabled ArgoCD to interact with our Managed Clusters.
From ACM Console, go to "Applications" then "Create" and select "ApplicationSet". You'll be prompted with a few fields to fill out, let's focus on the "Argo server" field. Click on it and then click on "Add Argo Server".
This will allow us to register our Managed Clusters inside the ArgoCD instance, by creating these 3 resources:
 - Placement resource: References our ManagedClusterSet
 - GitOpsCluster resource: Creates the "cluster" object inside our ArgoCD instance, based on the Placement resource
 - ManagedClusterSetBinding resource: Allows our openshit-gitops namespace to interact with the ManagedClusters in a specified ManagedClusterSet

Fill in the requested fields, and observe the above resources getting populated by toggling the "View Yaml" switch.

### Creating our demo app with GitOps
Now that we have our Managed Clusters registered into our ArgoCD instance, it's time to deploy our demo app on those.
Let's fill in the "name" fields, to provide a recognizable name for our application, any name will do.
Hit next, and you will be prompted with 2 choices: "Git" and "Helm", we will be using "Git" so click on it and fill in the fields as follows:
- URL: `https://github.com/agiorgirh/gitops-demoapp.git`
- Revision: `master`
- Path: `manifests`
- Remote namespace: `demoapp`

Click on next, you will be prompted with a few checkboxes that will configure the syncing and reconciliation mechanism for our application. We will leave everything as default here, and click next.
On this page we will define where our application will be deployed, indeed you will be prompted with "New Placement" or "Existing Placement". Go ahead and create a new placement rule to have it deployed on the "Prod" cluster only.
After you hit next again, you will see a review page, and a "Submit" button in the bottom. Go ahead and click it.

We have now deployed our demo app, which you can check out by navigating to "Applications", filter for "Application set" and you will see the app you just created.
If you click on it, you will get an Overview about the resources status, the last time it synced etc... Now go to the "Topology" tab, where you will be able to see your application resources getting created on the target cluster. If you click on the "Route" resources, and on the "Launch Route URL" you should be greeted with a Tetris-like webgame. 

Congrats, you deployed your app using GitOps on ACM!
