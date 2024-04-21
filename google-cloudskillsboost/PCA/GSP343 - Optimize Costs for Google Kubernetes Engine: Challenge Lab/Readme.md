# Optimize Costs for Google Kubernetes Engine: Challenge Lab (GSP343)

This document guides you through the GSP343 Challenge Lab on Qwiklabs, an advanced exercise designed to optimize costs in Google Kubernetes Engine (GKE). Throughout this lab, participants will engage in several key activities.

## Key Activities

### Deploying an Application on a Multi-Tenant Cluster
- Learn how to deploy your application in a multi-tenant environment, leveraging shared resources across different workloads.
- Migrating Cluster Workloads to an Optimized Node Pool
- Transition workloads to a node pool that has been optimized for cost and performance, enhancing the efficiency of resource usage.
- Rolling Out an Application Update While Maintaining Cluster Availability:
- Implement updates to your application without disrupting the ongoing services, ensuring high availability throughout the process.
- Cluster and Pod Autoscaling
- Explore the mechanisms of autoscaling, both at the cluster level and the pod level, to dynamically adjust to workload demands.

These tasks will help you enhance your skills in managing Kubernetes clusters efficiently while keeping costs in check.

## Guidelines for Deployment

### Task 1: Create Our Cluster and Deploy Our App

1. **Create the Cluster in the us-east4-c Zone.**
   - The naming scheme is `team-resource-number`, e.g., a cluster could be named `onlineboutique-cluster-642`.
   - For your initial cluster, start with machine size `e2-standard-2 (2 vCPU, 8G memory)`.
   - Set your cluster to use the rapid-release channel.
   - Create a new node pool named `optimized-pool-7139` with `custom-2-3584` as the machine type.

First, we will create a zone variable in the Cloud Shell. My Zone is **us-east4-c**.
```
ZONE=us-central1-a
```

Run the following to create the cluster with the specified configuration:
```
gcloud container clusters create onlineboutique-cluster-642 \
   --project=${DEVSHELL_PROJECT_ID} --zone=${ZONE} \
    --machine-type=e2-standard-2 --num-nodes=2
```

2. **Set Up Namespaces**
- After the GKE cluster is created, you need to set up the namespaces dev and prod.
```
kubectl create namespace dev
kubectl create namespace prod
```

3. **Deploy the Application**
- Deploy the application to the dev namespace. Switch the current namespace from default to dev:
```
kubectl config set-context --current --namespace dev
```

- Copy the application files to the Shell environment, and then deploy the OnlineBoutique app to GKE:
```
git clone https://github.com/GoogleCloudPlatform/microservices-demo.git &&
cd microservices-demo && kubectl apply -f ./release/kubernetes-manifests.yaml --namespace dev
```

- Monitor the service status using the following command:
```
kubectl get svc -w --namespace dev
```

- Locate the: ```frontend-external       LoadBalancer   10.10.76.49    34.86.151.158   80:31589/TCP   41s```
- _Press CTRL + C to stop the monitoring._ 
- Open http://34.86.151.158/ in a new tab. You should see the homepage of the Online Boutique application like this:


### **Task 2: Migrate to an Optimized Nodepool**

1. Create a New Node Pool
- Create a new node pool named optimized-pool-7139, configure custom-2-3584 as the machine type, and set the number of nodes to 2.
```
gcloud container node-pools create optimized-pool-7139 \
   --cluster=onlineboutique-cluster-642 \
   --machine-type=custom-2-3584 \
   --num-nodes=2 \
   --zone=$ZONE
```
Make sure you wait until the status of the new node pool becomes OK.

2. Migrate the Application
- Cordon and drain the default-pool:
```
for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl cordon "$node";
done
   
for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
   kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node";
done
```

- Verify if your pods are running on the new, `optimized-pool, node pool:
```
kubectl get pods -o=wide --namespace dev
```

With the pods migrated, it’s safe to delete the default-pool
```
gcloud container node-pools delete default-pool \
   --cluster onlineboutique-cluster-642 --zone $ZONE
```

### **Task 3: Apply a Frontend Update**
1. Set a Pod Disruption Budget
- Name it onlineboutique-frontend-pdb and set the min-availability of your deployment to 1.
```
kubectl create poddisruptionbudget onlineboutique-frontend-pdb \
--selector app=frontend --min-available 1  --namespace dev
```

2. Update Resources
- Use kubectl edit to update resources:
```
KUBE_EDITOR="nano -c" kubectl edit deployment/frontend --namespace dev
```

- Change the image to ```gcr.io/qwiklabs-resources/onlineboutique-frontend:v2.1``` and ```imagePullPolicy``` to ```Always``` line 58 and 59.
- Save the file changes.

### **Task 4: Autoscale from Estimated Traffic**

1. Configure the Autoscaler:
   - Scaling is based on a target CPU percentage of 50, and the pod scaling is set between 1 minimum and 11 maximum.
   - Apply horizontal pod autoscaling to your frontend deployment with:
```
kubectl autoscale deployment frontend --cpu-percent=50 \
   --min=1 --max=11 --namespace dev
```

- Check the status of the autoscalers by running:
```
kubectl get hpa --namespace dev
```

- Update your cluster autoscaler to scale between 1 node minimum and 6 nodes maximum.
```
gcloud beta container clusters update onlineboutique-cluster-642 \
   --enable-autoscaling --min-nodes 1 --max-nodes 6 --zone $ZONE
```

2. Test the Autoscalers
- Perform a load test to simulate the traffic surge:
```
kubectl exec $(kubectl get pod --namespace=dev | grep 'loadgenerator' | cut -f1 -d ' ') \
   -it --namespace=dev -- bash -c "export USERS=8000; sh ./loadgen.sh"
```
- Navigate to the OVERVIEW tab of the frontend deployment.
- You should see a sharp increase in CPU, Memory, and Disk utilization after the load test started, and the number of pods increasing by the horizontal pod autoscaling.
- Scroll to the Managed Pods section, you should observe the number of pods increases by the horizontal pod autoscaling.

### **Task 5. (Optional) Optimize other services**
While implementing horizontal pod autoscaling on your frontend service helps maintain availability during the load test, monitoring your other workloads will reveal that some are significantly straining for certain resources.
You can also see if it would be possible to further optimize your resource utilization with Node Auto Provisioning.

1. Try to enable node auto provisioning. This allows cluster autoscaler to create new node pools with different node specifications.
- --max-cpu refers to the max number of cores (e.g. 12) to which the cluster can scale to,
- --max-memory refers to the max number of gigabytes of memory (e.g. 8) to which the cluster can scale to.
```
gcloud container clusters update onlineboutique-cluster-642 \
    --enable-autoprovisioning --max-cpu=4 --max-memory=5 --zone $ZONE
```

2. “Optimize-utilization” for the autoscaler profile is likely NOT an option for a production e-commerce website that may encounter spiky traffic. It is more suitable for batch workloads.
```
gcloud container clusters update onlineboutique-cluster-642 \
    --autoscaling-profile=optimize-utilization --zone $ZONE
```

3. Vertical Pod Autoscaling (VPA) will not give accurate results for this lab as VPA requires time (at least 24 hours recommended) to collect usage data. Relevant command:
```
gcloud container clusters update onlineboutique-cluster-642 \
    --enable-vertical-pod-autoscaling --zone $ZONE
```
