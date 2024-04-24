# Set Up and Configure a Cloud Environment in Google Cloud- Challenge Lab (GSP321)

# Challenge scenario
As a cloud engineer at Jooli Inc. and recently trained with Google Cloud and Kubernetes, you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.
You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.
You need to complete the following tasks:

Create a development VPC with three subnets manually
Create a production VPC with three subnets manually
Create a bastion that is connected to both VPCs
Create a development Cloud SQL Instance and connect and prepare the WordPress environment
Create a Kubernetes cluster in the development VPC for WordPress
Prepare the Kubernetes cluster for the WordPress environment
Create a WordPress deployment using the supplied configuration
Enable monitoring of the cluster
Provide access for an additional engineer
Some Jooli Inc. standards you should follow:

Create all resources in the **REGION** region and **ZONE** zone, unless otherwise directed.
Use the project VPCs.
Naming is normally team-resource, e.g. an instance could be named k**raken-webserver1**.
Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use **e2-medium**.

Your challenge
You need to help the team with some of their initial work on a new project. They plan to use WordPress and need you to set up a development environment. Some of the work was already done for you, but other parts require your expert skills.

As soon as you sit down at your desk and open your new laptop you receive the following request to complete these tasks. Good luck!

![image](https://github.com/rianelis/GCP/assets/31323104/266e7cc6-c722-4743-8f0e-94db658b1997)


**My Lab Details:**
- Username 1: student-02-8bfaa70d62e0@qwiklabs.net
- Username 2: student-03-5dc0aeced461@qwiklabs.net (second account for Task 9)
- Project ID: qwiklabs-gcp-04-ccdde178c0a8
- Region: us-east1
- Zone : us-east1-c
- **Note: Replace with values provided to you.**

  
# Task 1. Create development VPC manually
Create a VPC called **griffin-dev-vpc** with the following subnets only:

- griffin-dev-wp
- IP address block: 192.168.16.0/20
- griffin-dev-mgmt
- IP address block: 192.168.32.0/20

- Activate Google Cloud Shell
```
gcloud auth list
```

-Set your Region and Zone
```
ZONE="us-east1-d"
REGION="us-east1"
```


- Create a VPC
```
gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region $region --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region $region --range=192.168.32.0/20
```

# Task 2. Create production VPC manually

- Create a VPC called griffin-prod-vpc with the following subnets only:
- griffin-prod-wp
- IP address block: 192.168.48.0/20
- griffin-prod-mgmt
- IP address block: 192.168.64.0/20

- Copy Configuration Files from Cloud Storage:
```
gsutil cp -r gs://cloud-training/gsp321/dm .
cd dm
```

- Edit the Configuration File:
```
sed -i s/SET_REGION/$region/g prod-network.yaml
```

- Create Deployment with Deployment Manager:
```
gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml
cd ..
```

# Task 3. Create bastion host

- Create a bastion host with two network interfaces, one connected to griffin-dev-mgmt and the other connected to griffin-prod-mgmt. Make sure you can SSH to the host.
  
```
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=$ZONE

gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc

gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc

```

# Task 4. Create and configure Cloud SQL Instance
- Create a MySQL Cloud SQL Instance called griffin-dev-db in us-east1.
- Connect to the instance and run the following SQL commands to prepare the WordPress environment:

- Creating the SQL Instance and set the root password: password
```
gcloud sql instances create griffin-dev-db --root-password password --region=$REGION --database-version=MYSQL_8_0
```

Connecting to the SQL Instance 
```
gcloud sql connect griffin-dev-db
```

- Enter root password: password
```
Connecting to database with SQL user [root].Enter password: password
```

- The GRANT ALL PRIVILEGES command in MySQL 8.0.
```
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
exit
```
** _Note: keep a record of the username and password you will need later on._ **


# Task 5. Create Kubernetes cluster
- Create a 2 node cluster (e2-standard-4) called griffin-dev, in the griffin-dev-wp subnet, and in zone ZONE.

```
gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone $ZONE
  
  ```
# **Task 6. Prepare the Kubernetes cluster**
- From Cloud Shell copy all files from gs://cloud-training/gsp321/wp-k8s. The WordPress server needs to access the MySQL database using the username and password you created in task 4.
- You do this by setting the values as secrets. WordPress also needs to store its working files outside the container, so you need to create a volume.
- Add the following secrets and volume to the cluster using wp-env.yaml.
- Make sure you configure the username to wp_user and password to stormwind_rules before creating the configuration.
- You also need to provide a key for a service account that was already set up. This service account provides access to the database for a sidecar container.
- Use the command below to create the key, and then add the key to the Kubernetes environment:


- Create a Service Account Key:
```
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

- Create a Kubernetes Secret:
```
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

- Copying and Configuring the WordPress Kubernetes Manifests:
```
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
cd wp-k8s
```

- Edit the WordPress Configuration File:
```
sed -i s/username_goes_here/wp_user/g wp-env.yaml
sed -i s/password_goes_here/stormwind_rules/g wp-env.yaml
```
***Note: Task 6 will be green after you complete task 7.**

  
# Task 7. Create a WordPress deployment

- Create Environment Config in Kubernetes:
```
kubectl create -f wp-env.yaml
```

- Create Service Account Key:
```
gcloud iam service-accounts keys create key.json --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

- Create Kubernetes Secret:
```
kubectl create secret generic cloudsql-instance-credentials --from-file key.json
```

- Get SQL Instance Connection Name:
```
I=$(gcloud sql instances describe griffin-dev-db --format="value(connectionName)")
```
* Note: Retrieves the connection name of a Cloud SQL instance, which is necessary for the Cloud SQL Proxy to connect to the correct instance.

- Edit Deployment Configuration File:
```
sed -i s/YOUR_SQL_INSTANCE/$I/g wp-deployment.yaml
```
* Note: Replaces a placeholder in the wp-deployment.yaml file with the actual Cloud SQL instance connection name.

- Deploy WordPress and Services:
```
kubectl create -f wp-deployment.yaml
kubectl create -f wp-service.yaml
```

# Task 8. Enable monitoring
- Create an uptime check for your WordPress development site.
- Go back to the Cloud Console, navigate to Kubernetes Engine, then click Workload.
- Click on wordpress and copy the IP address: 34.138.151.89
- Go back to the Cloud Console and navigate to Monitoring.
- In the Monitoring console, click Uptime checks in the left pane.
- Click CREATE UPTIME CHECK.
- Configure using the following parameters:
    Set the following values:
        Check TypeHTTP
        Resource TypeURL
        Hostname 34.138.151.89
        Path/
- Click CONTINUE
- Click Review and add Title: UptimeCheck
- Click TEST
- Click CREATE

# Task 9. Provide access for an additional engineer
You have an additional engineer starting and you want to ensure they have access to the project, so please go ahead and grant them the editor role to the project.
The second user account for the lab represents the additional engineer.

- In the Cloud Console, navigate to IAM & Admin > IAM.
- Click the PENCIL icon in the secodn user: student-03-5dc0aeced461@qwiklabs.net.
- In the Role dropdown, select Project > Editor.
- Click SAVE.

Congratulations!
