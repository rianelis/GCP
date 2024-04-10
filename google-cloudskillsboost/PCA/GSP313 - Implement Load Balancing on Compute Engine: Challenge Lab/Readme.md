## Lab Name: Create and Manage Cloud Resources: Challenge Lab (GSP313)


## Task 1: Create a project jumphost instance
Run command:

```
gcloud compute instances create nucleus-jumphost-941 \
          --network default \
          --zone us-west3-a\
          --machine-type e2-micro  \
          --image-family debian-12-bookworm-v20240312 \
          --image-project debian-cloud \
          --scopes cloud-platform \
          --no-address

```       
## Task 2: Create a Kubernetes service cluster
Run command:

```
gcloud container clusters create nucleus-backend \
          --num-nodes 1 \
          --network default \
          --region us-west3
```
```
gcloud container clusters get-credentials nucleus-backend \
          --region us-west3
```

```
kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0
```

```
kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port 8080
```

## Task 3: Set up an HTTP load balancer
Run command:

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
```
gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network default \
          --machine-type e2-micro\
          --region us-west3
```
```
gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region us-west3
```
```
gcloud compute firewall-rules create permit-tcp-rule-608 \
          --allow tcp:80 \
          --network default
```         
```          
gcloud compute http-health-checks create http-basic-check
```
```
gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region us-west3
```
```
gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global
``` 
```
gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-region us-west3\
          --global
```
```
gcloud compute url-maps create web-server-map \
          --default-service web-server-backend
```
```
gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map
```
```
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```
```        
gcloud compute forwarding-rules list
```
