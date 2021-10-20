# Build and Secure Networks in Google Cloud: Challenge Lab

## Task 1: Remove the overly permissive rules

```bash
gcloud compute firewall-rules delete open-access
```

## Task 2: Start the `bastion` vm instance

```bash
gcloud compute instances start bastion --zone=us-central1-b
```

## Task 3: Create firewall rule to allow ssh to `bastion`

```bash
gcloud compute instances add-tags bastion --zone=us-central1-b --tags=bastion
gcloud compute firewall-rules create allow-ssh-iap --direction=INGRESS --target-tags=bastion --network=acme-vpc --allow=tcp:22 --source-ranges=35.235.240.0/20
```

## Task 4: Create firewall rule to allow http traffic on port 80 to `juice-shop`

```bash
gcloud compute instances add-tags juice-shop --zone=us-central1-b --tags=juice-shop
gcloud compute firewall-rules create allow-http-juice-shop --allow=tcp:80 --target-tags=juice-shop --network=acme-vpc
```

## Task 5: Allow ssh to `juice-shop` in `acme-mgmt-subnet`

```bash
gcloud compute networks subnets list
gcloud compute firewall-rules create allow-ssh-bastion-juice-shop --target-tags=bastion,juice-shop --source-ranges=<acme-mgmt-subnet ip address> --allow=tcp:22 --network=acme-vpc
```

## Task 6: SSH to `bastion`, then SSH to `juice-shop` from `bastion`

```bash
ssh <juice-shop internal ip>
```
