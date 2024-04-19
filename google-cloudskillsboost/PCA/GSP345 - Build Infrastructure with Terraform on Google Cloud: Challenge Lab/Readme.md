## Automating Infrastructure on Google Cloud with Terraform: Challenge Lab (GSP345) ##

***Topics tested***:
Import existing infrastructure into your Terraform configuration.
Build and reference your own Terraform modules.
Add a remote backend to your configuration.
Use and implement a module from the Terraform Registry.
Re-provision, destroy, and update infrastructure.
Test connectivity between the resources you've created.


***My details:***

- Project ID: qwiklabs-gcp-03-9e7a9808ef9a
- Bucket Name: tf-bucket-606613
- Instance Name: tf-instance-477404
- VPC Name: tf-vpc-566985
- Region: europe-west1
- Zone: europe-west1-c


<br/> **Task 1. Create the configuration files** <br/>
Make the empty files and directories in _Cloud Shell_ or the _Cloud Shell Editor_.

```
touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf
touch outputs.tf
touch variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf
touch outputs.tf
touch variables.tf
cd

```

Fill out the variables.tf files in the root directory and within the modules. Add three variables to each file: region, zone, and project_id. For their default values, use europe-west1, europe-west1-c, and your Google Cloud Project ID (qwiklabs-gcp-03-9e7a9808ef9a). Be aware you might probably have different regions, zones and project IDs.
```
variable "region" {
 default = "europe-west1"
}

variable "zone" {
 default = "europe-west1-c"
}

variable "project_id" {
 default = "qwiklabs-gcp-03-9e7a9808ef9a"
}

```

Add the following to the _main.tf_ file:

```
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0" #">= 3.55.0, < 6.0.1"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}


module "instances" {
  source     = "./modules/instances"
}

```
Run "_terraform init_" in Cloud Shell in the root directory to initialize terraform.
```
terraform init

```
<br/> **TASK 2: Import infrastructure** <br/>

Navigate to _Compute Engine > VM Instances_. Click on _tf-instance-1_. Copy the _Instance ID_ down somewhere to use later. <br/>
Navigate to _Compute Engine > VM Instances_. Click on _tf-instance-2_. Copy the _Instance ID_ down somewhere to use later. <br/>
Next, navigate to _modules/instances/instances.tf_. Copy the following configuration into the file:

```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }

metadata_startup_script = <<-EOT
#!/bin/bash
EOT

allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }

metadata_startup_script = <<-EOT
#!/bin/bash
EOT

allow_stopping_for_update = true
}

```
To import the first instance, use the following command, using the Instance ID for **_tf-instance-1_** you copied down earlier. My Instance ID is **5404001346111886902**
```
terraform import module.instances.google_compute_instance.tf-instance-1 5404001346111886902

```
To import the second instance, use the following command, using the Instance ID for **_tf-instance-2_** you copied down earlier. My Instance ID is **4168654664566003254**
```
terraform import module.instances.google_compute_instance.tf-instance-2 4168654664566003254

```
The two instances have been imported into your Terraform configuration. You can now run the commands to update Terraform's state. After you run the apply command, type _yes_ in the dialogue to accept the state changes.
```
terraform plan
terraform apply
```

**Note: Plan: 0 to add, 2 to change, 0 to destroy.**

Next, navigate to _modules/instances/instances.tf_. Copy the following configuration into the file on both instances:
```
  network_interface {
 network = "default"
  }

metadata_startup_script = <<-EOT
#!/bin/bash
EOT

allow_stopping_for_update = true
}
```
```
terraform plan
terraform apply
```
**Note: Plan: 2 to add, 0 to change, 2 to destroy.**
**Note: This extra step is due to lab circumstances. Without this step, the check progress will not be successful until Task 6. This step destroys the current instances and recreates them with the metadat_startup_script**

<br/> **TASK 3: Configure a remote backend** <br/>
Add the following code to the **_modules/storage/storage.tf_** file, and fill in the _Bucket Name_: My bucket name is **tf-bucket-606613**

```
resource "google_storage_bucket" "tf-bucket-606613" {
  name          = "tf-bucket-606613"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}

```
Next, add the following to the _main.tf_ file:
```
module "storage" {
  source     = "./modules/storage"
}

```
Run the following commands to initialize the module and create the storage bucket resource. Type _yes_ in the dialogue after you run the apply command to accept the state changes.
```
terraform init
terraform apply

```
Next, update the _main.tf_ file so that the terraform block looks like the following. Fill in your _GCP Project ID_ for the bucket argument definition. My project id is qwiklabs-gcp-03-9e7a9808ef9a
```
terraform {
  backend "gcs" {
    bucket  = "tf-bucket-606613"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}

```
Run the following to initialize the remote backend. Type _yes_ at the prompt.
```
terraform init
```
<br/> **TASK 4: Modify and update infrastructure** <br/>
Navigate to _modules/instances/instance.tf_. Replace the entire contents of the file with the following, and fill in your _Instance 3 ID_: My 3rd instance name is tf-instance-477404.

```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = "europe-west1-c"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = "europe-west1-c"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-477404" {
  name         = "tf-instance-477404"
  machine_type = "e2-standard-2"
  zone         = "europe-west1-c"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }
}

```
Run the following commands to initialize the module and create/update the instance resources. Type _yes_ in the dialogue after you run the apply command to accept the state changes.

```
terraform init
terraform apply
```
<br/> **TASK 5: Taint and destroy resources** <br/>
Taint the _tf-instance-3_ resource by running the following command, and fill in your _Instance 3 ID_: My Instance 3 is tf-instance-477404.

```
terraform taint module.instances.google_compute_instance.tf-instance-477404

```
Run the following commands to apply the changes:
```
terraform init
terraform apply
```
Remove the _tf-instance-3_ resource from the _instances.tf_ file. Delete the following code chunk from the file.
```
resource "google_compute_instance" "tf-instance-477404" {
  name         = "tf-instance-477404"
  machine_type = "e2-standard-2"
  zone         = "europe-west1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }
}
```
Run the following commands to apply the changes. Type _yes_ at the prompt.

```
terraform apply

```
<br/> **TASK 6: Use a module from the Registry** <br/>
Change the required_providers ">= 3.55.0, < 6.0.0" in _main.tf_ file

```
terraform {
  backend "gcs" {
    bucket  = "tf-bucket-606613"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 3.55.0, < 6.0.1"
    }
  }
}

```

Copy and paste the following to the end of _main.tf_ file, fill in _Version Number_ and _Network Name_ instructed in the challenge: My version is 6.0.0 and my network is tf-vpc-566985.

```

module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "qwiklabs-gcp-03-9e7a9808ef9a"
    network_name = "tf-vpc-566985"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "europe-west1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "europe-west1"
        }
    ]
}

```
Run the following commands to initialize and upgrade the module and create the VPC. Type _yes_ at the prompt.
```
terraform init -upgrade
terraform apply

```
Navigate to _modules/instances/instances.tf_. Replace the entire contents of the file with the following:

```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = "europe-west1-c"
 

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "tf-vpc-566985"
    subnetwork = "subnet-01"
  }
  metadata_startup_script = <<-EOT
  #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = "europe-west1-c"
  

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "tf-vpc-566985"
    subnetwork = "subnet-02"
  }

  metadata_startup_script = <<-EOT
  #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```
Run the following commands to initialize the module and update the instances. Type _yes_ at the prompt.

```
terraform init
terraform apply

```
<br/> **TASK 6: Configure a firewall** <br/>
Add the following resource to the _main.tf_ file, fill in the _GCP Project ID_ and _Network Name_: My project is is qwiklabs-gcp-03-9e7a9808ef9a and my network name is tf-vpc-566985

```
resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
 network = "projects/qwiklabs-gcp-03-9e7a9808ef9a/global/networks/tf-vpc-566985"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}

```
Run the following commands to configure the firewall. Type _yes_ at the prompt.
```
terraform init
terraform apply

```
