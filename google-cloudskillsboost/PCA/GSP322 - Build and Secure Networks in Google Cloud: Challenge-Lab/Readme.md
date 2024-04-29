Username
student-00-3e3efafb2569@qwiklabs.net

Project ID
qwiklabs-gcp-03-b5be6f96ad58

SSH IAP network tag
permit-ssh-iap-ingress-ql-367

SSH internal network tag
permit-ssh-internal-ingress-ql-367

HTTP network tag
permit-http-ingress-ql-367

Zone
us-east1-b (Make sure this matches the zone specified in your lab instructions and with the zone the VM are running.)




export ZONE=us-east1-b
echo $ZONE


# Task 1 : Remove the overly permissive rules:

gcloud compute firewall-rules delete open-access



# Task 2 : Start the bastion host instance:

# Go to Compute Engine and start Bastion instance.



# Task 3 : Create a firewall rule that allows SSH (tcp/22) from the IAP service and add network tag on bastion:

# Replace the "accept-ssh-iap-ingress-ql-716" with the network tag provided in the lab. 

gcloud compute firewall-rules create ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags accept-ssh-iap-ingress-ql-367 --network acme-vpc

gcloud compute instances add-tags bastion --tags=accept-ssh-iap-ingress-ql-367 --zone=$ZONE



# Task 4 : Create a firewall rule that allows traffic on HTTP (tcp/80) to any address and add network tag on juice-shop:
# Replace the "accept-http-ingress-ql-716" with the network tag provided in the lab. 

gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags accept-http-ingress-ql-367 --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=accept-http-ingress-ql-367 --zone=$ZONE



# Task 5 : Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address and add network tag on juice-shop:
# Replace the "accept-ssh-internal-ingress-ql-716" with the network tag provided in the lab. 

gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags accept-ssh-internal-ingress-ql-367 --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=accept-ssh-internal-ingress-ql-367 --zone=$ZONE



# Task 6 : SSH to bastion host via IAP and juice-shop via bastion:

# In Compute Engine -> VM Instances page, click the SSH button for the bastion host. 
# Then SSH to juice-shop with command:

ssh 192.168.11.2