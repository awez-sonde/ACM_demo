# RHACM Workshop 


## Access the console 

Once logged in, navigate to the MultiCluster Switcher and select All Clusters to see the ACM UI.


Now, navigate to the Clusters screen, you will notice that the Hub cluster is already in the list as a managed cluster (local-cluster), we will use this later in the guide.



## Cluster Lifecycle (Cluster Lifecycle Management use case)

At a high level Cluster Lifecycle management is about creating, upgrading, and destroying and importing clusters in a multi cloud environment.

Its time to use your  AWS credentials, Access Key ID, Secret Access Key, and the Base DNS Domain.  In order to create a new OpenShift cluster in the AWS cloud we will need these keys to create a Provider Connection. On the left bar, select Credentials and then select Add Credential.

You will need to provide connection details
- Credential Type: Choose Amazon Web Services and then, Amazon Web Services again
- Credential Name:  aws
- Namespace: open-cluster-management
- Base DNS Domain:  your_base_dn
- Click NEXT
- Access Key ID:  your-access-id
- Secret Access Key ID: your-secret-id
- Click NEXT - We don’t need to configure a Proxy
- Red Hat OpenShift pull secret:  Get from your [Red Hat login](https://cloud.redhat.com/openshift/install/pull-secret)
- SSH private and public keys:  Use an existing key pair or generate a new one (see link below) 
- Click NEXT

Verify the information and click ADD

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



## cqs4CwwiABFhCiZzJ9U30HUQ1hVgxp

## https://dev141762.service-now.com






