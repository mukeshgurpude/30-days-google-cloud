# Create and Manage Cloud Resources: Challenge Lab

Url: <https://www.cloudskillsboost.google/focuses/10258>

Quest: <https://www.cloudskillsboost.google/quests/120>

First setup `zone` and `region` as per lab requirements

```bash
gcloud config set compute/zone us-east1-b
gcloud config set compute/region us-east1
```

## Task 1

This task requires creation of simple Virtual Machine using given name and machine type. Ensure this VM is created in `us-east1-b` zone.

```bash
gcloud compute instances create nucleus-jumphost \
--machine-type f1-micro
```

## Task 2

- Create a cluster to build application on kubernetes

  ```bash
  gcloud container clusters create nucleus-webserver
  gcloud container clusters get-credentials nucleus-webserver
  ```

- Create a sample deployment using image given in the instructions. Then expose a `loadBalancer` service on port `8080`.

  ```bash
  kubectl create deployment hello-app --image gcr.io/google-samples/hello-app:2.0

  kubectl expose deployment hello-app --type LoadBalancer --port 8080
  ```

## Task 3

In this task, you're required to configure load balancing for ngix web server

- Copy the given startup script in a file(`startup.sh`). This will install and start `nginx` in each instance created using the template.

  ```bash
  cat << EOF > startup.sh
  #! /bin/bash
  apt-get update
  apt-get install -y nginx
  service nginx start
  sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
  EOF
  ```

- Create instance template. Use startup script file from previous step.

  ```bash
  gcloud compute instance-templates create nginx-template \
  --tags allow-health-check \
  --image-family debian-9 \
  --image-project debian-cloud \
  --metadata-from-file startup-script=startup.sh
  ```

- Create a target pool.

  ```bash
  gcloud compute target-pools create nginx-pool
  ```

- Create a managed instance group. This group shall use `nginx-template`, and `nginx-pool`. After creation of group, set a named port, which will be used by our `backend-service`

  ```bash
  gcloud compute instance-groups managed create nginx-group \
    --template nginx-template \
    --base-instance-name nginx \
    --size 2 \
    --target-pool nginx-pool

  gcloud compute instance-groups managed \
    set-named-ports nginx-group \
    --named-ports http:80
  ```

  > You'll get 10 points in Task 3 at this point

- Create a firewall rule to allow traffic at (80/tcp).

  ```bash
  gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
  ```

  > You'll get 15 points in Task 3 at this point

- Create a health check.

  ```bash
  gcloud compute health-checks create http http-basic-check \
  --port 80
  ```

- Create a backend service, and attach the managed instance group.

  ```bash
    gcloud compute backend-services create nginx-backend-service \
      --protocol=HTTP \
      --port-name=http \
      --health-checks=http-basic-check \
      --global
  ```

  ```bash
    gcloud compute backend-services add-backend nginx-backend-service \
      --instance-group=nginx-group \
      --instance-group-zone=us-east1-b \
      --global
  ```

  > You'll get 25 points in Task 3 at this point

- Create a URL map, and target the HTTP proxy to route requests to your URL map.

  ```bash
    gcloud compute url-maps create nginx-map-http \
        --default-service nginx-backend-service
  ```

  > You'll get 30 points in Task 3 at this point

  ```bash
    gcloud compute target-http-proxies create nginx-proxy \
        --url-map nginx-map-http
  ```

  > You'll get 35 points in Task 3 at this point

- Create a forwarding rule.

  Create a static ip address globally

  ```bash
  gcloud compute addresses create nginx-ip --ip-version=IPV4 --global
  ```

  Use above ip address for our forwarding rule

  ```bash
    gcloud compute forwarding-rules create http-content-rule \
        --address=nginx-ip\
        --global \
        --target-http-proxy=nginx-proxy \
        --ports=80
  ```

  > You'll get 40 points in Task 3 at this point

- Verify Load balancers are working. execute following commands to list down forwarding-rules

  ```bash
  gcloud compute forwarding-rules list
  ```

  Copy exerternal ip for `http-content-rule`, and enter in the browser. Wait for this page to load. It may take 2-3 minutes to configure it properly.
  
  As soon as, load balancer starts working, you'll get `100/100` for the lab
