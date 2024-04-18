# Deploy and Manage Cloud Environments with Google Cloud: Challenge Lab - GSP314

## Task 1: Migrate a stand-alone PostgreSQL database to a Cloud SQL for PostgreSQL instance

### Enable APIs

# Enable Database Migration API 
# Enable Service Networking API

# Upgrade the database with the pglogical extension
```
sudo apt install postgresql-13-pglogical
```
# Download and apply configuration for pglogical
```
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"
```
# Restart PostgreSQL service
```
sudo systemctl restart postgresql@13-main
```
# Launch the psql tool
```
sudo su - postgres
psql
```

# Add the pglogical extension
```
\c postgres;
CREATE EXTENSION pglogical;
\c orders;
CREATE EXTENSION pglogical;
```
# Create the database migration user
```
CREATE USER import_admin PASSWORD 'DMS_1s_cool!';
ALTER DATABASE orders OWNER TO import_admin;
ALTER ROLE import_admin WITH REPLICATION;
```

# Grant permissions to the migration user
```
\c postgres;
GRANT USAGE ON SCHEMA pglogical TO import_admin;
GRANT ALL ON SCHEMA pglogical TO import_admin;
GRANT SELECT ON ALL TABLES IN SCHEMA pglogical TO import_admin;

\c orders;
GRANT USAGE ON SCHEMA public TO import_admin;
GRANT ALL ON SCHEMA public TO import_admin;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO import_admin;

\dt
ALTER TABLE public.distribution_centers OWNER TO import_admin;
ALTER TABLE public.inventory_items OWNER TO import_admin;
ALTER TABLE public.order_items OWNER TO import_admin;
ALTER TABLE public.products OWNER TO import_admin;
ALTER TABLE public.users OWNER TO import_admin;
\dt

ALTER TABLE public.inventory_items ADD PRIMARY KEY(id);
\q 
Exit
```


#### Create a new connection profile for the PostgreSQL source instance.  have the follwing details, you will need to replacce them with the ones provide to you.

**Username:** import_admin
**Password:** DMS_1s_cool!
**Region:** us-east4

1. Navigate to Database Migration > Connection profiles.
2. Click + Create Profile.
3. Fill in the form:
   - Database engine: PostgreSQL
   - Connection profile name: postgres-vm
   - Hostname or IP address: 10.150.0.2 (example IP)
   - Port: 5432
   - Username: import_admin
   - Password: DMS_1s_cool!
   - Region: us-east4
4. Click Create.

#### Create a new continuous migration job
1. Navigate to Database Migration > Migration jobs.
2. Click + Create Migration Job.
3. Configure the migration job. I have the follwing details, you will need to replacce them with the ones provide to you:
   - Migration job name: b2b-postgres21
   - Source database engine: PostgreSQL
   - Destination database engine: Cloud SQL for PostgreSQL
   - Migration job type: Continuous
   - Destination region: us-east4
4. Click Save & Continue.

#### Promote Cloud SQL instance
1. Navigate to the migration job details.
2. Click Promote to finalize the migration.


## Task 2: Update permissions and add IAM roles to users
### Permissions and IAM Roles
1. Grant the Antern Editor user the Cloud SQL Instance User role for the CloudSQL database. Username: ```student-00-1c5e37ec9606@qwiklabs.net```.
2. Add the Antern Editor user account to the Cloud SQL database with Cloud IAM authentication. Use their username mentioned above.
3. Grant the Cymbal Owner user the Cloud SQL Admin role for the CloudSQL database. Username: ```student-04-c3aaa887b485@qwiklabs.net```
4. Add the Cymbal Owner user account to the Cloud SQL database with Cloud IAM authentication using their username.
5. Change the Cymbal Editor user role from Viewer to Editor. Username: ```student-03-e7eb3cfacd76@qwiklabs.net```

## Task 3: Create networks and firewalls

```
export VPC_NAME=vpc-network-nsgf
export SUBNET_A=subnet-a-smaz
export REGION_A=us-west1
export SUBNET_B=subnet-b-e9ku
export REGION_B=us-east
export VPC_NAME=vpc-network-nsgf
export SUBNET_A=subnet-a-smaz
export REGION_A=us-west1
export SUBNET_B=subnet-b-e9ku
export REGION_B=us-east4
export FIREWALL_RULE_NAME_1=ebpe-firewall-ssh
export FIREWALL_RULE_NAME_2=ecdb-firewall-rdp
export FIREWALL_RULE_NAME_3=gnwo-firewall-icmp

gcloud compute networks create $VPC_NAME --project=$DEVSHELL_PROJECT_ID --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional 
gcloud compute networks subnets create $SUBNET_A --project=$DEVSHELL_PROJECT_ID --range=10.10.10.0/24 --stack-type=IPV4_ONLY --network=$VPC_NAME --region=$REGION_A 
gcloud compute networks subnets create $SUBNET_B --project=$DEVSHELL_PROJECT_ID --range=10.10.20.0/24 --stack-type=IPV4_ONLY --network=$VPC_NAME --region=$REGION_B

gcloud compute --project=$DEVSHELL_PROJECT_ID firewall-rules create $FIREWALL_RULE_NAME_1 --direction=INGRESS --priority=65535 --network=$VPC_NAME --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute --project=$DEVSHELL_PROJECT_ID firewall-rules create $FIREWALL_RULE_NAME_2 --direction=INGRESS --priority=65535 --network=$VPC_NAME --action=ALLOW --rules=tcp:3389 --source-ranges=0.0.0.0/0
gcloud compute --project=$DEVSHELL_PROJECT_ID firewall-rules create $FIREWALL_RULE_NAME_3 --direction=INGRESS --priority=65535 --network=$VPC_NAME --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0
```

## ask 4: Troubleshoot and fix a broken GKE cluster

### Create a BigQuery log sink
1. In the Cloud console, select Navigation menu > Logging > Log Router.
2. Click Create sink.
3. Set up the sink:
   - Name: frontend-service-error-sink
   - Destination: BigQuery dataset named gke_app_errors_sink with a location of US.
   - Inclusion filter: `resource.type=frontend-service-error-sink; severity=ERROR`
4. Click Create.

### Assign roles
Grant the Antern Editor user the BigQuery Data Viewer role. Username: Antern Editor username.
Grant the Antern Owner user the BigQuery Admin role. Username: Antern Owner username.

