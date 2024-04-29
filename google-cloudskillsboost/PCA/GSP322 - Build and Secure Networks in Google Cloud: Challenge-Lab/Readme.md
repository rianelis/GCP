# Build a Secure Google Cloud Network: Challenge Lab (GSP322)

## Challenge scenario
You are a security consultant brought in by Jeff, who owns a small local company, to help him with his very successful website (juiceshop). Jeff is new to Google Cloud and had his neighbour's son set up the initial site. The neighbour's son has since had to leave for college, but before leaving, he made sure the site was running.

Below is the current set up:

![image](https://github.com/rianelis/GCP/assets/31323104/f5b4d5af-018d-452b-afc7-f16c2ae92b68)

## Your challenge
You need to create the appropriate security configuration for Jeff's site. Your first challenge is to set up firewall rules and virtual machine tags. You also need to ensure that SSH is only available to the bastion via IAP.

For the firewall rules, make sure that:

- The bastion host does not have a public IP address.
- You can only SSH to the bastion and only via IAP.
- You can only SSH to juice-shop via the bastion.
- Only HTTP is open to the world for juice-shop.

Tips and tricks:

- Pay close attention to the network tags and the associated VPC firewall rules.
- Be specific and limit the size of the VPC firewall rule source ranges.
- Overly permissive permissions will not be marked correct.

## The Google Cloud environment to configure

![image](https://github.com/rianelis/GCP/assets/31323104/f1a7a1bf-f3dd-42d5-a2ce-14f95741a910)

## Lab Details Assigned:

- Username: student-00-3e3efafb2569@qwiklabs.net
- Project ID: qwiklabs-gcp-03-b5be6f96ad58
- SSH IAP network tag: permit-ssh-iap-ingress-ql-950
- SSH internal network tag: permit-ssh-internal-ingress-ql-950
- HTTP network tag: permit-http-ingress-ql-950
- Zone: us-east1-d (Make sure this matches the zone specified in your lab instructions and with the zone the VM are running.)
- baston VM: IP - 192.168.10.2
- juice-shop VM: IP - 192.168.11.2 & 34.23.248.66


## Suggested order of action.

```
export ZONE=us-east1-d
echo $ZONE
```

## Task 1: Remove the overly permissive rules:
```
gcloud compute firewall-rules delete open-access
```


## Task 2: Start the bastion host instance: 
Go to Compute Engine and start Bastion instance. (Write down the IP addresses.)

## Task 3: Create a firewall rule that allows SSH (tcp/22) from the IAP service and add a network tag on bastion: 
Replace the **accept-ssh-iap-ingress-ql-716** with the network tag provided in the lab. To allow IAP to connect to your VM instances, create a firewall rule that allows ingress traffic from the IP range 35.235.240.0/20. This range contains all IP addresses that IAP uses for TCP forwarding.

```
gcloud compute firewall-rules create ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags accept-ssh-iap-ingress-ql-950 --network acme-vpc

gcloud compute instances add-tags bastion --tags=accept-ssh-iap-ingress-ql-950 --zone=$ZONE
```


## Task 4: Create a firewall rule that allows traffic on HTTP (tcp/80) to any address and add a network tag on juice-shop: 
Replace the **accept-http-ingress-ql-716** with the network tag provided in the lab. 

```
gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags accept-http-ingress-ql-950 --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=accept-http-ingress-ql-950 --zone=$ZONE
```

## Task 5: Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address and add a network tag on juice-shop:
Replace the **accept-ssh-internal-ingress-ql-716** with the network tag provided in the lab. 

```
gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags accept-ssh-internal-ingress-ql-950 --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=accept-ssh-internal-ingress-ql-950 --zone=$ZONE
```

## Task 6: SSH to bastion host via IAP and juice-shop via bastion:
- In the Compute Engine -> VM Instances page, click the SSH button for the bastion host. (Write down the IP addresses.)
- Then SSH to juice-shop with the command:
  
```
ssh 192.168.11.2
```
