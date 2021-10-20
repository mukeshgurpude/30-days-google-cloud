# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab

## Quick setup

- Set default `zone` and `region`

  ```bash
  gcloud config set compute/region us-east1
  gcloud config set compute/zone us-east1-b
  ```

## Task 1: Create development VPC manually

Create a VPC called `griffin-dev-vpc` with the following subnets only:

- name: `griffin-dev-wp`
  ip range: `192.168.16.0/20`
- name: `griffin-dev-mgmt`
  ip range: `192.168.32.0/20`

```bash
gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network griffin-dev-vpc --range 192.168.16.0/20
gcloud compute networks subnets create griffin-dev-mgmt --network griffin-dev-vpc --range 192.168.32.0/20
```

## Task 2: Create production VPC manually

Create a VPC called `griffin-prod-vpc` with the following subnets only:

- name: `griffin-prod-wp`
  ip range: `192.168.48.0/20`
- name: `griffin-prod-mgmt`
  ip range: `192.168.64.0/20`

```bash
gcloud compute networks create griffin-prod-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-prod-wp --network griffin-prod-vpc --range 192.168.48.0/20
gcloud compute networks subnets create griffin-prod-mgmt --network griffin-prod-vpc --range 192.168.64.0/20
```

## Task 3: Create bastion host

Go to Navigation Menu -> Compute Engine -> Instances and create a new instance named `bastion-host`. Use `n1-standard-1` machine type. Refer _Create a VM instance with multiple network interfaces_ section in [Multiple VPC Networks Lab](https://www.cloudskillsboost.google/focuses/1230?parent=catalog) to add below networks to the instance...

```bash
gcloud compute firewall-rules create allow-ssh-dev --network=griffin-dev-vpc --rules=tcp:22 --action=ALLOW
gcloud compute firewall-rules create allow-ssh-prod --network=griffin-prod-vpc --rules=tcp:22 --action=ALLOW
```

## Task 4: Create and configure Cloud SQL Instance

Create a sql instance with named `griffin-dev-db`...

```bash
gcloud sql instances create griffin-dev-db --database-version=MYSQL_5_7 --root-password=password
```

```bash
gcloud sql connect griffin-dev-db
```

```bash
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```

## Task 5: Create Kubernetes cluster

```bash
gcloud container clusters create griffin-dev --subnetwork griffin-dev-wp --network griffin-dev-vpc --num-nodes 2
gcloud container clusters get-credentials griffin-dev
```

## Task 6: Prepare the Kubernetes cluster

- Copy sample files from bucket

  ```bash
  gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
  cd wp-k8s
  ```

- Create secret keys, for access to the database. First edit `wp-env.yaml`, and replace username and password with `wp_user` and `stormwind_rules`. Now create environment secret using below commands

  ```bash
  kubectl create -f wp-env.yaml
  gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
  kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
  ```

## Task 7: Create a WordPress deployment

- Use below command to print out sql instance details

  ```bash
  gcloud sql instances describe griffin-dev-db
  ```

  copy down the instance `connectionName`.
- In `wp-deployments.yaml`, replace `YOUR_SQL_INSTANCE` with _connectionName_ copied above
- Create deployment and service

  ```bash
  kubectl create -f wp-deployment.yaml
  kubectl create -f wp-service.yaml
  ```

## Task 8: Enable monitoring

From Navigation Menu navigate to *Monitoring -> Uptime Checks*
Click on _Create uptime check_

- Get IP address of `wordpress-dev` service

  ```bash
  kubectl get services
  ```

- Give a title - `wp-uptime-check`
- In Target, enter `external ip` of wp-dev Loadbalancer as `hostname`. and `/` in path.
- Click `test` to test the connection, and create the uptime check

## Task 9: Provide access for an additional engineer

- Navigate to _Navigation Menu -> Iam and Admin_
- `Add` a new account
- copy second username from the connection panels into `new principals` field
- For permissions, add `Project -> Editor`, and save the details
