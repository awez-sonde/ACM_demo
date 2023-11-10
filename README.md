# RHACM Workshop 


## Access the console 

Once logged in, navigate to the MultiCluster Switcher and select All Clusters to see the ACM UI.


Now, navigate to the Clusters screen, you will notice that the Hub cluster is already in the list as a managed cluster (local-cluster), we will use this later in the guide.


## Cluster Lifecycle (Cluster Lifecycle Management use case)

At a high level Cluster Lifecycle management is about creating, upgrading, and destroying and importing clusters in a multi cloud environment.

Its time to use your  AWS credentials, Access Key ID, Secret Access Key, and the Base DNS Domain.  In order to create a new OpenShift cluster in the AWS cloud we will need these keys to create a Provider Connection. On the left bar, select Credentials and then select Add Credential.
