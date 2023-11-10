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

| Hub cluster provisioned                                  | Managed cluster region |
| ----- | ----- |
| If you provisioned your ACM Hub cluster in NORTH AMERICA | Select us-west-1 or us-west-2                        |
| If you provisioned your ACM Hub cluster in EUROPE / EMEA | Select eu-west-2 or eu-west-3                        |
| If you provisioned your ACM Hub cluster in ASIA PACIFIC  | Select ap-southeast-2 or ap-northeast-2 or ap-east-1 |

---
**NOTE**

Please note that the deployment might still fail, if so please retry at different availability zone, we have very limited amount of elastic IP addresses in these accounts.

---

